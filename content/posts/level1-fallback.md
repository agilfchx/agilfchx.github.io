---
title: "Level 1 - Fallback"
date: 2023-08-18T23:14:53+07:00
draft: false
tags: ["Ethernaut"]
categories: ["Smart Contract Securiy"]
---

Hello Readers!

Kita akan melanjutkan series Ethernaut ini dan kita akan bermain di level 1 yaitu `Fallback`

Pada level kali ini tidak seperti sebelumnya kita langsung diberikan sebuah source code smart contract nya untuk menyelesaikan level ini, berikut adalah source codenya

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {

  mapping(address => uint) public contributions;
  address public owner;

  constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }

  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```

> Goals untuk menyelesaikan level ini adalah :

1. Kita sebagai `player` harus claim `ownership` contract tersebut
2. Mengurangi balancenya ke 0/withdraw semua amount yang ada di contract tersebut
   >

### Breakdown the code

Oke mari kita breakdown setiap code yang ada pada source code tersebut satu persatu

```solidity
mapping(address => uint) public contributions;        <= 1
address public owner;                                 <= 2
```

Pada contract `Fallback` ini terdapat 2 variable yaitu:

1. `contributions` yang bertipe data `mapping` , yang dimana mapping ini akan menyimpan tipe data ke tipe data lainnya. Disini dia akan memetakan/mapping `address` ke tipe data `uint` atau simple nya data bertipe `uint` akan disimpan pada `address`
2. `owner` merupakan pemilik dari contract ini

```solidity
  constructor() {
    owner = msg.sender;                                <= 1
    contributions[msg.sender] = 1000 * (1 ether);      <= 2
  }
```

Pada `constructor` kita breakdown seperti berikut

1. Variable `owner` akan di set oleh `msg.sender` pertama atau orang yang melakukan deploy contract tersebut.
2. `contributions[msg.sender]` ini menetapkan nilai kontribusi `msg.sender` sebesar 1000 Ether (1000 \* 1 ether) dalam satuan wei

```solidity
  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }
```

Modifer `onlyOwner` ini untuk memastikan bahwa function tersebut hanya bisa dijalankan oleh `owner` itu sendiri (`msg_sender == owner`) dan jika bukan owner akan muncul output `caller is not the owner`. Dan terakhir `_;` merupakan tanda untuk melanjutkan jika modifier ini sudah diperiksa dan sesuai.

```solidity
  function contribute() public payable {
    require(msg.value < 0.001 ether);                           <= 1
    contributions[msg.sender] += msg.value;                     <= 2
    if(contributions[msg.sender] > contributions[owner]) {      <= 3
      owner = msg.sender;                                       <= 4
    }
  }
```

Selanjutnya terdapat function `contribute` yang dimana dapat menerima Ether (payable), berikut nya pada body functionnya ini terdapat:

1. `require` disini bertujuan untuk memeriksa apakah nilai Ether yang diberikan dalam transaksi kurang dari 0.001 Ether, jika tidak memenuhi maka perintah dibawahnya tidak akan dijalankan/transaksi gagal.
2. `contributions[msg.sender]` akan menambahkan nilai Ether yang diterima dari transaksi ke address pengirim
3. Kondisi ini mengecek apakah kontribusi `msg.sender` lebih besar dibanding milik `owner`
4. Jika kondisi tersebut `true` maka ownership contract akan dignati menjadi pengirim atau `msg.sender` yang melakukan kontribusi

```solidity
function getContribution() public view returns (uint) {
   return contributions[msg.sender];
}
```

Pada function `getContribution` ini cukup simple, dia akan melakukan return address dari `msg.sender` berupa `uint` atau data berapa besar dia kontribusi.

```solidity
function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
 }
```

Pada function `withdraw` ini akan melakukan withdraw tetapi dengan syarat dia adalah `onlyOwner` atau `owner` dari contract tersebut.

```solidity
  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
```

`receive()` merupakan function yang mirip dengan fallback function tetapi receive ini bertujuan untuk menangani ether yang masuk tanpa perlu panggilan data. Disini dia melakukan pemeriksaan atau `require` seperti apakah `msg.value` nya lebih dari 0 DAN `contributions[msg.sender]` memiliki kontribusi lebih dari 0 dan jika kondisi tersebut terpenuhi maka `msg.sender` atau caller ini akan menjadi `owner` dari contract ini.

### Explainoit (Explain & Exploit)

Oke dari source code yang didapatkan kita mendapatkan beberapa informasi yang bisa membantu kita menyelesaikan level ini, yaitu:

1. Pada function `contribute` kita bisa mengubah ownership contract dengan cara `value` kontribusi kita harus lebih besar dari owner yaitu 1000 Ether 😮
2. Kita juga bisa menjadi ownership contract di `receive` function yang dimana terdapat sebuah require seperti **_value yang dikirimkan lebih dari 0_** dan **_kontribusi lebih dari 0_** juga

Dari informasi ke 2 ini kita dapat melakukan merubah ownership contract tanpa perlu mengeluarkan ether yang banyak. Intinya adalah kita akan melakukan hit/interaksi ke `receive` function langsung karena kita bisa menjadi ownership contract juga dari function fallback tersebut.

Sebelum melakukan exploit mari kita cari tahu dahulu address owner contract ini. Untuk melihat siapa owner contract ini bisa menjalankan perintah berikut ini di console

```solidity
await contract.owner()
```

Dan kita lihat bahwa address owner contract sekarang adalah `0x2279B7A0a67DB372996a5FaB50D91eAA73d2eBe6`

{{< figure align=center src="/images/20230818/owner.png" title="Initial Owner Address" >}}

Selanjutnya, karena `receive()` memerlukan require salah satunya `contributions[msg.sender] > 0`

kita perlu menjalankan `contribute()` terlebih dahulu untuk membuat kontribusi, berikut perintah untuk melakukan kontribusi dan nanti akan muncul popup metamask untuk konfirmasi transaksi

```solidity
await contract.contribute({ value: 1 })
// untuk value gunakan saja yang terendah, seperti diatas yaitu hanya 1 wei
```

{{< figure align=center src="/images/20230818/contribute.png" title="Running await contract.contribute() in console" >}}

Lalu kita cek dahulu apakah contribution kita sudah masuk atau belum dengan menjalankan perintah berikut

```solidity
await contract.getContribution().then(v => v.toString())
```

{{< figure align=center src="/images/20230818/getcontribution.png" title="Running await contract.getContribution() in console" >}}

Selanjutnya, untuk melakukan interaksi langsung ke `fallback` atau `receive` kita bisa menjalankan perintah berikut ini dan akan muncul popup metamask seperti biasa untuk konfirmasi

```solidity
await contract.sendTransaction({ from: player, value: 1 })
```

{{< figure align=center src="/images/20230818/sendtransac.png" title="Running await contract.sendTransaction() in console" >}}

Sekarang jika kita lihat owner contractnya sudah berubah sesuai dengan address yang kita miliiki

{{< figure align=center src="/images/20230818/owner-final.png" title="Owner contract address changed to our address" >}}

Langkah terakhir adalah melakukan withdraw dengan menjalankan perintah berikut ini dan kalian bisa lihat balance kalian bertambah

```solidity
await contract.withdraw()
```

{{< figure align=center src="/images/20230818/withdraw.png" title="Running await contract.withdraw() in console" >}}

Setelah selesai lakukan `Submit Instance` dann level 1 pun selesai

Oke level 1 sudah mulai menantang karena kita mencari code yang memungkinkan bisa kita exploit untuk mendapatkan owner dari contract itu sendiri. Baiklah kalau begitu mungkin segitu saja untuk level 1 ini, kita akan lanjutkan kembali level 2 dan seterusnya 🔥

Seeya 👋🐢
