# Rakamin-KF-Analytics
-- Memmbuat tabel baru ‘kf_final_data_mart' pada dataset kimia_farma berdasarkan query
CREATE TABLE kimia_farma.kf_final_data_mart AS

WITH data_mart AS (
  -- Mengambil data dari tabel 'kf_final_transaction', 'kf_kantor_cabang', dan 'kf_product'
  SELECT
    ft.transaction_id,
    ft.date,
    ft.branch_id,
    kc.branch_name,
    kc.kota,
    kc.provinsi,
    kc.rating AS rating_cabang,
    ft.customer_name,
    ft.product_id,
    p.product_name,
    ft.price AS actual_price,
    ft.discount_percentage,

  -- Menentukan persentase gross laba berdasarkan harga
  CASE
    WHEN ft.price <= 50000 THEN 0.1
    WHEN ft.price > 50000 AND ft.price <= 100000 THEN 0.15
    WHEN ft.price > 100000 AND ft.price <= 300000 THEN 0.2
    WHEN ft.price > 300000 AND ft.price <= 500000 THEN 0.25
    ELSE 0.3
  END AS percentage_gross_laba,

  -- Menghitung penjualan bersih setelah diskon
  (ft.price - (ft.price * ft.discount_percentage)) AS nett_sales,

  ft.rating AS rating_transaksi
FROM kimia_farma.kf_final_transaction AS ft

-- Melakukan LEFT JOIN dengan tabel 'kf_kantor_cabang' untuk mendapatkan detail kantor cabang
LEFT JOIN kimia_farma.kf_kantor_cabang kc ON ft.branch_id = kc.branch_id

-- Melakukan LEFT JOIN dengan tabel kf_product untuk mendapatkan detail product
LEFT JOIN kimia_farma.kf_product p ON ft.product_id = p.product_id
)

-- Memasukkan data hasil dari CTE ke dalam tabel yang baru dibuat
SELECT DISTINCT
  dm.*,
  -- Menghitung laba bersih
  ROUND (dm.nett_sales * dm.percentage_gross_laba, 2) AS nett_profit
FROM data_mart AS dm;
