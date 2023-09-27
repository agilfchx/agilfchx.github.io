---
title: "Level 5 - Token"
date: 2023-09-27T23:53:17+07:00
draft: false
tags: ["Ethernaut"]
categories: ["Smart Contract Securiy"]
---

Haloo readers, kali ini kita akan ngelanjutin series Ethernaut ini dari sela kesibukan ngoprek sana sini. Lesgo kita mulai

Seperti biasa kita dikasih source codenya seperti berikut

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

> Goal: Mendapatkan token tambahan (melebihi 20 token)
> Hint: odometer

### Breaking the code

Terdapat sebuah function yang memungkinkan bisa kita exploit yaitu di `transfer()` yang dimana memiliki parameter `address _to` dan `uint _value` lalu dia akan mereturn sebuah `bool`

```solidity
 function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);              <= 1
    balances[msg.sender] -= _value;                           <= 2
    balances[_to] += _value;                                  <= 3
    return true;
  }
```

- Bagian `1` terdapat require yang dimana _balance_ kita jika dikurangi `*_value`\* harus lebih besar dari 0.
- Bagian `2` akan mengurangi amount token atau `_value` yang akan di transfer dari balance kita
- Bagian `3` akan memasukkan token atau `_value` ke recipient/penerima balance

### Explainoit (Explain & Exploit)

Jadi pada kode yang diberikan ini terdapat beberapa hal yang perlu diperhatikan. Seperti yang kita tahu bahwa `_value` di function `transfer()` ini memiliki tipe data `uint` dan pada bagian `1` di _breaking the code_, dijelaskan bahwa terdapat require/pengecekkan bahwa kita tidak bisa mengirim token lebih dari yang kita miliki.

Sebelum memasuki ke exploitnya sebaiknya kita mengetahui apa maksud dari hint **_odometer_** ini, jadi maksud dari hint ini adalah ketika _kondisi angka sudah berada di batas maksimal maka dia akan kembali ke 0_ (overflow) dan jika ada overflow maka ada underflow yang dimana _kondisi angka sudah ada di posisi 0 dan dikurangi 1 maka dia akan mengganti valuenya ke byte tertingginya_.

Berikut adalah gambaran mengenai overflow dan underflow menggunakan tipe data `uint8` yang dimana maksimal bytenya adalah `255` dari [0xSage](https://medium.com/coinmonks/ethernaut-lvl-5-walkthrough-how-to-abuse-arithmetic-underflows-and-overflows-2c614fa86b74)

{{< figure align=center src="/images/20230927/Untitled.png" title="overflow dan underflow pada integer" >}}

Kembali lagi ke function `transfer()` kita lihat bahwa dia menggunakan tipe data `uint` pada `_value` nya dan pada smart contract ini menggunakan Solidity 0.6 yang dimana versi ini masih **_belum secure_** terhadap [overflow dan underflow](https://hackernoon.com/hack-solidity-integer-overflow-and-underflow), maka bisa kita pastikan bahwa level ini bisa kita exploit dengan underflow

Sebelum itu kita akan cek terlebih dahulu berapa token kita sekarang

```jsx
await contract.balanceOf(player).then((v) => v.toString());
```

{{< figure align=center src="/images/20230927/Untitled 1.png" title="token saat ini" >}}

Kita memiliki 20 Token untuk saat ini, selanjutnya kita akan transfer dengan valuenya melebihi token kita katakan 21 Token dan untuk penerimanya kita akan menggunakan null address yaitu `0x0000000000000000000000000000000000000000`

```jsx
await contract.transfer("0x0000000000000000000000000000000000000000", 21);
```

Setelah itu kita cek kembali balance kita dengan perintah sebelumnya

{{< figure align=center src="/images/20230927/Untitled 2.png" title="cek balance" >}}

Yepp balance kita bertambah, Submit Instance dan level 5 selesai. Lalu kita diberikan sebuah note seperti berikut

{{< figure align=center src="/images/20230927/Untitled 3.png" title="note tentang overflow" >}}

Jadi untuk mencegah/pengecekkan overflow yang pertama kita bisa mengecek dengan code _control statement_ yang diberikan, lalu untuk alternative di semua mathematical operator kita bisa menggunakan `SafeMath` sehingga ketika ada overflow, maka code akan di revert.
