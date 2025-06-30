# Proyek Data Engineer: Pembangunan Data Warehouse & Solusi Analisis untuk ID/X Partners

Ini adalah repositori untuk proyek akhir program **Project-Based Internship Virtual Experience - Data Engineer** yang diselenggarakan oleh Rakamin Academy bekerja sama dengan ID/X Partners. Semua kode dan desain yang relevan disajikan dalam file tunggal ini untuk kemudahan peninjauan.

## 1. Latar Belakang & Tujuan Proyek

### Latar Belakang Masalah
Klien dari industri perbankan menghadapi tantangan signifikan dalam analisis data. Sumber data transaksi mereka tersebar di berbagai format dan lokasi, termasuk database SQL Server, file CSV, dan file Excel. Proses konsolidasi data yang dilakukan secara manual menyebabkan keterlambatan dalam pelaporan, yang pada akhirnya menghambat pengambilan keputusan bisnis yang cepat dan akurat.

### Tujuan Proyek
Tujuan utama dari proyek ini adalah merancang dan mengimplementasikan solusi data yang efisien dengan:
1.  **Membangun Data Warehouse (DWH):** Membuat sebuah sumber kebenaran tunggal (*single source of truth*) yang terstruktur untuk mengintegrasikan semua data.
2.  **Merancang Pipeline ETL:** Mendesain alur kerja untuk proses Ekstraksi, Transformasi, dan Pemuatan (ETL) data ke dalam DWH.
3.  **Mengembangkan Alat Analisis:** Membuat Stored Procedure di SQL untuk menyediakan ringkasan data penting secara cepat dan otomatis.

---

## 2. Pembangunan Data Warehouse (Skema & Kode SQL)

Data Warehouse dirancang menggunakan *Star Schema* untuk mengoptimalkan query analitik. Struktur ini terdiri dari satu tabel fakta (`FactTransaction`) dan tiga tabel dimensi (`DimAccount`, `DimCustomer`, `DimBranch`).

### Kode SQL untuk Skema DWH
Berikut adalah skrip T-SQL lengkap yang digunakan untuk membuat database dan semua tabel yang diperlukan.

```sql
/*
================================================================
Skrip Pembuatan Skema Data Warehouse
================================================================
*/

-- Membuat Database jika belum ada
IF NOT EXISTS (SELECT * FROM sys.databases WHERE name = 'DWH')
BEGIN
    CREATE DATABASE DWH;
END
GO

USE DWH;
GO

-- 1. Tabel Dimensi: DimAccount
CREATE TABLE DimAccount (
    AccountID INT PRIMARY KEY,
    CustomerID INT,
    AccountType VARCHAR(50),
    Balance MONEY,
    DateOpened DATE,
    Status VARCHAR(20)
);
GO

-- 2. Tabel Dimensi: DimBranch
CREATE TABLE DimBranch (
    BranchID INT PRIMARY KEY,
    BranchName VARCHAR(100),
    BranchLocation VARCHAR(255)
);
GO

-- 3. Tabel Dimensi: DimCustomer
CREATE TABLE DimCustomer (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100),
    Address VARCHAR(255),
    CityName VARCHAR(100),
    StateName VARCHAR(100),
    Age INT,
    Gender VARCHAR(50),
    Email VARCHAR(100)
);
GO

-- 4. Tabel Fakta: FactTransaction
CREATE TABLE FactTransaction (
    TransactionID INT PRIMARY KEY,
    AccountID INT,
    TransactionDate DATETIME,
    Amount MONEY,
    TransactionType VARCHAR(50),
    BranchID INT,
    FOREIGN KEY (AccountID) REFERENCES DimAccount(AccountID),
    FOREIGN KEY (BranchID) REFERENCES DimBranch(BranchID)
);
GO
```

---

## 3. Desain Pipeline ETL Menggunakan Talend

Pipeline ETL dirancang di Talend Open Studio untuk menangani proses pemindahan dan transformasi data dari sumber ke DWH.

### Desain Job untuk Memuat Tabel Dimensi
Contoh desain job untuk memuat data ke `DimCustomer`, yang melibatkan join data dan transformasi format.
![Desain Job Dimensi](https://github.com/[NAMA_USER_ANDA]/[NAMA_REPO_ANDA]/blob/main/assets/Desain_Job_Load_DimCustomer.png)

### Desain Job untuk Memuat Tabel Fakta
Desain job ini menangani konsolidasi data dari tiga sumber berbeda (database, CSV, Excel), membersihkan duplikat, dan memuatnya ke `FactTransaction`.
![Desain Job Fakta](https://github.com/[NAMA_USER_ANDA]/[NAMA_REPO_ANDA]/blob/main/assets/Desain_Job_Load_FactTransaction.png)

---

## 4. Alat Analisis Otomatis (Stored Procedures)

Dua Stored Procedure dikembangkan untuk memberikan insight bisnis secara cepat.

### Stored Procedure `DailyTransaction`
Menghasilkan laporan harian yang merangkum jumlah transaksi dan total nominalnya.

```sql
CREATE OR ALTER PROCEDURE DailyTransaction
    @start_date DATE,
    @end_date DATE
AS
BEGIN
    SET NOCOUNT ON;
    SELECT
        CAST(TransactionDate AS DATE) AS [Date],
        COUNT(TransactionID) AS TotalTransactions,
        SUM(Amount) AS TotalAmount
    FROM
        FactTransaction
    WHERE
        CAST(TransactionDate AS DATE) BETWEEN @start_date AND @end_date
    GROUP BY
        CAST(TransactionDate AS DATE)
    ORDER BY
        [Date];
END
GO
```

### Stored Procedure `BalancePerCustomer`
Menghitung saldo akhir (Current Balance) nasabah berdasarkan transaksi yang terjadi.

```sql
CREATE OR ALTER PROCEDURE BalancePerCustomer
    @name VARCHAR(100)
AS
BEGIN
    SET NOCOUNT ON;
    WITH TransactionChanges AS (
        SELECT
            AccountID,
            SUM(CASE WHEN TransactionType = 'Deposit' THEN Amount ELSE -Amount END) AS NetChange
        FROM FactTransaction
        GROUP BY AccountID
    )
    SELECT
        c.CustomerName,
        a.AccountType,
        a.Balance AS InitialBalance,
        (a.Balance + ISNULL(tc.NetChange, 0)) AS CurrentBalance
    FROM
        DimCustomer c
    JOIN DimAccount a ON c.CustomerID = a.CustomerID
    LEFT JOIN TransactionChanges tc ON a.AccountID = tc.AccountID
    WHERE
        c.CustomerName LIKE '%' + @name + '%' AND a.Status = 'active';
END
GO
```

## Status Proyek & Teknologi

*   **Status:** Perancangan dan pengembangan DWH serta Stored Procedure telah **selesai**. Desain pipeline ETL juga telah **selesai**. Eksekusi fungsional ETL tertunda karena kendala konektivitas spesifik yang sedang dalam proses *troubleshooting*.
*   **Teknologi:** `SQL Server (T-SQL)`, `SSMS`, `Talend Open Studio`.

---
*Dibuat oleh [Nama Anda]*
