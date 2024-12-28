# query-riski-anugerah # Nama Proyek

Membuat tabel analisa dari hasil agregasi dataset dan tabel yang sudah dilakukan


## Penjelasan dan Syntaxnya
--langkah membuat table analisa 


1. --Membuat kolom id transaksi sampai diskon persentase dahulu
SELECT
    t.transaction_id,                      -- ID Transaksi
    t.date,                                -- Tanggal Transaksi
    t.branch_id,                           -- ID Cabang
    b.branch_name,                         -- Nama Cabang
    b.kota,                                -- Kota Cabang
    b.provinsi,                            -- Provinsi Cabang
    b.rating AS rating_cabang,             -- Penilaian Cabang
    t.customer_name,                       -- Nama Customer
    t.product_id,                          -- ID Produk
    p.product_name,                        -- Nama Produk
    p.price AS actual_price,               -- Harga Produk
    t.discount_percentage                  -- Persentase Diskon
FROM
    `rakamin-kf-analytics-445713.kimia_farma.kf_final_transaction` t
JOIN
    `rakamin-kf-analytics-445713.kimia_farma.kf_product` p ON t.product_id = p.product_id
JOIN
    `rakamin-kf-analytics-445713.kimia_farma.kf_kantor_cabang` b ON t.branch_id = b.branch_id


--Penjelasan Query:
--1.	Penggabungan (JOIN):
--kf_final_transaction (tabel transaksi) digabungkan dengan kf_product (tabel produk) berdasarkan product_id, sehingga dapat mengambil nama produk dan harga.
--kf_final_transaction juga digabungkan dengan kf_kantor_cabang berdasarkan branch_id, sehingga  bisa mengambil nama cabang, kota, provinsi, dan rating cabang.
--2.	Kolom yang Diambil:
--Kolom-kolom seperti transaction_id, date, branch_id, branch_name, kota, provinsi, rating_cabang, customer_name, product_id, product_name, actual_price (harga produk), dan discount_percentage diambil dari tabel yang sesuai.




2--membuat kolom persentase gross laba
--•	Kolom persentase_gross_laba dihitung menggunakan CASE statement:
--WHEN p.price <= 50000 THEN 10: Jika harga produk kurang dari atau sama dengan Rp 50.000, maka laba adalah 10%.
--WHEN p.price > 50000 AND p.price <= 100000 THEN 15: Jika harga produk lebih dari Rp 50.000 dan kurang dari atau sama dengan Rp 100.000, maka laba adalah 15%.
--WHEN p.price > 100000 AND p.price <= 300000 THEN 20: Jika harga produk lebih dari Rp 100.000 dan kurang dari atau sama dengan Rp 300.000, maka laba adalah 20%.
--WHEN p.price > 300000 AND p.price <= 500000 THEN 25: Jika harga produk lebih dari Rp 300.000 dan kurang dari atau sama dengan Rp 500.000, maka laba adalah 25%.
--WHEN p.price > 500000 THEN 30: Jika harga produk lebih dari Rp 500.000, maka laba adalah 30%.
--ELSE 0: Jika harga produk tidak memenuhi kondisi di atas, laba akan dianggap 0%.

SELECT
    t.transaction_id,                      -- ID Transaksi
    t.date,                                -- Tanggal Transaksi
    t.branch_id,                           -- ID Cabang
    b.branch_name,                         -- Nama Cabang
    b.kota,                                -- Kota Cabang
    b.provinsi,                            -- Provinsi Cabang
    b.rating AS rating_cabang,             -- Penilaian Cabang
    t.customer_name,                       -- Nama Customer
    t.product_id,                          -- ID Produk
    p.product_name,                        -- Nama Produk
    p.price AS actual_price,               -- Harga Produk
    t.discount_percentage,                -- Persentase Diskon
    -- Menghitung Persentase Gross Laba Berdasarkan Harga
    CASE
        WHEN p.price <= 50000 THEN 10
        WHEN p.price > 50000 AND p.price <= 100000 THEN 15
        WHEN p.price > 100000 AND p.price <= 300000 THEN 20
        WHEN p.price > 300000 AND p.price <= 500000 THEN 25
        WHEN p.price > 500000 THEN 30
        ELSE 0
    END AS persentase_gross_laba
FROM
    `rakamin-kf-analytics-445713.kimia_farma.kf_final_transaction` t
JOIN
    `rakamin-kf-analytics-445713.kimia_farma.kf_product` p ON t.product_id = p.product_id
JOIN
    `rakamin-kf-analytics-445713.kimia_farma.kf_kantor_cabang` b ON t.branch_id = b.branch_id




3--menambahkan kolom nett sales, profit dan rating dan query final

SELECT
    t.transaction_id,                      -- ID Transaksi
    t.date,                                -- Tanggal Transaksi
    t.branch_id,                           -- ID Cabang
    b.branch_name,                         -- Nama Cabang
    b.kota,                                -- Kota Cabang
    b.provinsi,                            -- Provinsi Cabang
    b.rating AS rating_cabang,             -- Penilaian Cabang
    t.customer_name,                       -- Nama Customer
    t.product_id,                          -- ID Produk
    p.product_name,                        -- Nama Produk
    p.price AS actual_price,               -- Harga Produk
    t.discount_percentage,                -- Persentase Diskon
    -- Menghitung Persentase Gross Laba Berdasarkan Harga
    CASE
        WHEN p.price <= 50000 THEN 10
        WHEN p.price > 50000 AND p.price <= 100000 THEN 15
        WHEN p.price > 100000 AND p.price <= 300000 THEN 20
        WHEN p.price > 300000 AND p.price <= 500000 THEN 25
        WHEN p.price > 500000 THEN 30
        ELSE 0
    END AS persentase_gross_laba,
    -- Menghitung Harga Setelah Diskon (Nett Sales)
    (p.price * (1 - t.discount_percentage / 100)) AS nett_sales,
    -- Menghitung Keuntungan Kimia Farma (Nett Profit)
    ((p.price * (1 - t.discount_percentage / 100)) * 
     CASE
        WHEN p.price <= 50000 THEN 10
        WHEN p.price > 50000 AND p.price <= 100000 THEN 15
        WHEN p.price > 100000 AND p.price <= 300000 THEN 20
        WHEN p.price > 300000 AND p.price <= 500000 THEN 25
        WHEN p.price > 500000 THEN 30
        ELSE 0
    END / 100) AS nett_profit,
    -- Rating Transaksi
    t.rating AS rating_transaksi
FROM
    `rakamin-kf-analytics-445713.kimia_farma.kf_final_transaction` t
JOIN
    `rakamin-kf-analytics-445713.kimia_farma.kf_product` p ON t.product_id = p.product_id
JOIN
    `rakamin-kf-analytics-445713.kimia_farma.kf_kantor_cabang` b ON t.branch_id = b.branch_id

