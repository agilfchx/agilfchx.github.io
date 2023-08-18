---
title: "Setup Ethernaut"
date: 2023-08-18T23:14:30+07:00
draft: false
tags: ["Ethernaut"]
categories: ["Smart Contract Securiy"]
---

Halo semuanya, jadi pada post pertama ini saya akan membagikan cara untuk setup Ethernaut secara lokal (Linux).

Jadi sebelum kita melakukan setup, saya akan membagikan beberapa versi tools dan software yang saya gunakan untuk bermain di Ethernaut yang dapat berjalan di lokal saya yaitu:

1. Node.js v18.17.1
2. NPM v9.8.1
3. yarn
4. git

## Steps

1. Clone repo dan install dependencies

```bash
git clone https://github.com/OpenZeppelin/ethernaut.git && yarn install
```

2. Setelah terinstall, jalankan `ganache` atau rpc servernya pada folder `ethernaut`

```bash
yarn network
```

{{< figure align=center src="/ardiaswx/images/20230818/yarn-network.png" >}}

3. Lanjut kita akan melakukan installasi `Metamask` sesuai browser yang digunakan
4. Setelah `Metamask` terinstall, selanjutnya kita akan menambahkan network baru dengan klik pada bagian pojok kiri atas lalu klik `Add Network` dan setelah di klik pilih `Add Network Manually`

{{< figure align=center src="/ardiaswx/images/20230818/add-network.png" >}}

5. Isi kolom yang kosong menjadi seperti berikut dan untuk `Network name` bebas diisi sesuai kemauan kalian

{{< figure align=center src="/ardiaswx/images/20230818/set-network.png" >}}

6. Setelah itu kita akan memasukkan akun yang dari Ganache, jadi kita memerlukan private key yang ada pada output Ganache di terminal

{{< figure align=center src="/ardiaswx/images/20230818/priv-key.png">}}

7. Untuk Import Account yang ada di Ganache ini kita bisa klik `Account 1` atau yang bagian atas yang paling tengah, lalu akan nanti akan muncul sebuah poppup dan pilih `Import account`

{{< figure align=center src="/ardiaswx/images/20230818/imp-acc.png" >}}

8. Masukkan private key yang sudah di copy dari Ganache ke kolom berikut ini, lalu Import

{{< figure align=center src="/ardiaswx/images/20230818/imp-priv.png" >}}

Tampilan jika berhasil melakukan import account

{{< figure align=center src="/ardiaswx/images/20230818/success-imp.png" >}}

9. Setelah `Metamask` ready, kita akan pindah untuk melakukan deploy dan menjalankan servernya nanti. Tetapi sebelumnya kita pergi ke file `client/src/constant.js` untuk melakukan uncomment pada `line 180` agar site nya bisa dijalankan menggunakan `NETWORK.LOCAL`

{{< figure align=center src="/ardiaswx/images/20230818/uncomment-180.png" >}}

10. Pindah lagi ke atas yaitu di `line 16` yaitu `url` dari `localhost` menjadi `http://127.0.0.1`

{{< figure align=center src="/ardiaswx/images/20230818/change-url.png" >}}

> Ini terjadi error karena gatau kenapa gabisa deploy pake `localhost` 🫤

11. Langsung saja deploy contract dengan menjalankan perintah sebagai berikut

```bash
yarn deploy:contracts
```

Nanti akan muncul `Confirm deployment` dan bisa ketik `y` saja

{{< figure align=center src="/ardiaswx/images/20230818/conf-depl.png" >}}

Berikut tampilan jika deploy contract berhasil

{{< figure align=center src="/ardiaswx/images/20230818/success-depl.png" >}}

12. Sebelum menjalankan server/clientnya, kita perlu menambahkan dahulu pada file `package.json` di bagian script `start` dan `build`nya (melanjutkan saja) karena terdapat error juga

```javascript
export SET NODE_OPTIONS=--openssl-legacy-provider &&
```

{{< figure align=center src="/ardiaswx/images/20230818/change-pkg.png" >}}

13. Baru kita bisa memulai server/client dengan menjalankna perintah berikut ini

```bash
yarn start:ethernaut
```

14. Jika kita baru menjalankan pertama kali maka nanti akan muncul popup dari `Metamask` seperti berikut untuk memilih akun yang telah kita import dari ganache agar bisa terhubung ke site Ethernaut lokal kita.

{{< figure align=center src="/ardiaswx/images/20230818/con-mt.png" >}}

Selanjutnya tinggal pilih `Connect` saja

{{< figure align=center src="/ardiaswx/images/20230818/connect.png" >}}

15. Nanti jika sudah selesai maka akan muncul sebuah popup seperti berikut, cukup klik `Deploy game`

{{< figure align=center src="/ardiaswx/images/20230818/deploy-game.png" >}}

16. Dan terakhir akan muncul sebuah popup dari `Metamask` untuk konfirmasi transaksinya

{{< figure align=center src="/ardiaswx/images/20230818/conf-mt.png" >}}

Yep setelah itu kita tinggal memainkan level mana saja yang kita inginkan.

> Sebenarnya setup seperti ini tidak efektif karena pasti bakal terjadi error. **Jadi perlu diingat** untuk memulai level ini kembali lakukan **deploy kembali** dan **ganti account** di Metamask menggunakan **private key yang lain** jangan sama seperti sebelumnya.

Mungkin segitu saja pada post pertama saya ini yaitu melakukan setup Ethernaut agar bisa dijalankan secara lokal.

Semoga perjalanan saya untuk belajar Smart Contract Audit dan menyampaikan solusi dari game Ethernaut ini atau tulisan lainnya dapat membantu kalian 😁

See you next post 👋
