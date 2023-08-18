---
title: "Level 0 - Hello Ethernaut"
date: 2023-08-18T23:14:44+07:00
draft: false
tags: ["Ethernaut"]
categories: ["Smart Contract Securiy"]
---

Hee ya 👋

Pada post kali ini saya akan memulai dari Level 0 yaitu `Hello Ethernaut`

Di level ini kita akan diberitahu bagaimana cara bermain di Ethernaut, tanpa panjang lebar mari kita mulai saja

## Start

Untuk memulai kita klik `Get new instance` terlebih dahulu untuk memulai level ini. Nanti akan muncul sebuah pop up confirm transaction dari `Metamask` dan itu bisa langsung saja klik Confirm.

> Jika terjadi error seperti berikut ini, coba import account menggunakan private key lain dan jangan lupa di connect an ke site nya
> {{< figure align=center src="/ardias/images/20230818/error1.png" >}}

Setelah klik `Get new instance` maka tampilan nya akan berubah dan akan ada button `Submit instance`.
{{< figure align=center src="/ardias/images/20230818/submit-instance.png" >}}

Kalau kita buka console dan ketikkan `ethernaut` lalu muncul object seperti berikut maka instance berhasil di buat
{{< figure align=center src="/ardias/images/20230818/object.png" >}}

> Untuk melihat owner contract ethernaut yang dibuat / new instance yang tadi di klik bisa ketikkan
>
> ```jsx
> await ethernaut.owner();
> // menggunakan await agar tidak terjadi promise
> ```

Lalu bagaimana untuk menyelesaikan level ini? Jadi jika kita perhatikan pada nomor 9 untuk berinteraksinya kita perlu mengetikkan method `await contract.info()` untuk menyelesaikan level ini.

{{< figure align=center src="/ardias/images/20230818/info.png" >}}

Setelah menjalankan info ternyata diberi tahu kembali bahwa kita perlu mencari tahu di `info1()`. Pada `info1()` kita diberi tahu kembali bahwa disuruh mencoba `info2()` tetapi kita memerlukan parameter `"hello"`

{{< figure align=center src="/ardias/images/20230818/info1.png" >}}

Selanjutnya kita diberitahu bahwa terdapat property `infoNum` yang terdapat angka untuk `info` selanjutnya

{{< figure align=center src="/ardias/images/20230818/infonum.png" >}}

Kita mendapatkan angka untuk mengakses `info` selanjutnya yaitu `info42`

{{< figure align=center src="/ardias/images/20230818/info42.png" >}}

Dari `info42` ini kita dapat informasi lagi bahwa selanjutnya ada di method `theMethodName()`

{{< figure align=center src="/ardias/images/20230818/themethod.png" >}}

Disini kita mendapatkan method name lagi yaitu `method7123949()`

{{< figure align=center src="/ardias/images/20230818/method7123949.png" >}}

Selanjutnya kita perlu mencari tahu password untuk di submit ke function`authenticate()`

{{< figure align=center src="/ardias/images/20230818/pass-authenticate.png" >}}

Jika kita perhatikan `abi` yang ada di contract dengan mengetikkan `contract` terdapat sebuah function menarik yaitu `password`

{{< figure align=center src="/ardias/images/20230818/contract-pass.png" >}}

Setelah kita coba jalankan, kita mendapatkan password yaitu `ethernaut0`

{{< figure align=center src="/ardias/images/20230818/pass.png" >}}

Langsung saja kita selesaikan levelnya dengan memasukkan ke parameter function `authenticate()`

{{< figure align=center src="/ardias/images/20230818/auth.png" >}}

Nanti akan muncul sebuah popup Metamask lagi untuk Confirm transaksi dan jika berhasil pada console seperti berikut

{{< figure align=center src="/ardias/images/20230818/confirm.png" >}}

Untuk cek apakah level 0 sudah clear bisa ketikkan

{{< figure align=center src="/ardias/images/20230818/check.png" >}}

Dan pada tampilan klik button `Submit Instance`, setelah itu akan muncul popup confirm lagi. Tampilan akan menunjukkan Congratulations dan nanti akan muncul smart contract yang digunakan pada level ini

{{< figure align=center src="/ardias/images/20230818/submit.png" >}}

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Instance {

  string public password;
  uint8 public infoNum = 42;
  string public theMethodName = 'The method name is method7123949.';
  bool private cleared = false;

  // constructor
  constructor(string memory _password) {
    password = _password;
  }

  function info() public pure returns (string memory) {
    return 'You will find what you need in info1().';
  }

  function info1() public pure returns (string memory) {
    return 'Try info2(), but with "hello" as a parameter.';
  }

  function info2(string memory param) public pure returns (string memory) {
    if(keccak256(abi.encodePacked(param)) == keccak256(abi.encodePacked('hello'))) {
      return 'The property infoNum holds the number of the next info method to call.';
    }
    return 'Wrong parameter.';
  }

  function info42() public pure returns (string memory) {
    return 'theMethodName is the name of the next method.';
  }

  function method7123949() public pure returns (string memory) {
    return 'If you know the password, submit it to authenticate().';
  }

  function authenticate(string memory passkey) public {
    if(keccak256(abi.encodePacked(passkey)) == keccak256(abi.encodePacked(password))) {
      cleared = true;
    }
  }

  function getCleared() public view returns (bool) {
    return cleared;
  }
}
```

Gimana? Mudah bukan ... untuk level 0 nya 😝

Nah jadi seperti itu cara bermain di Ethernaut dan sebenarnya kita bisa gunakan berbagai cara ntah melalui `Dev Tools` seperti `console` atau jalanin langsung di `Remix IDE` atau menggunakan `foundry`.

Jadi untuk saat ini saya akan menggunakan `Dev Tools` atau `console` milik browser untuk menyelesaikan beberapa level yang ada di Ethernaut, lalu kita akan menggunakan `Remix IDE` untuk melakukan exploitasi terhadap smart contract yang ada di Ethernaut ini dan menyelesaikan levelnya.

Mungkin jika sudah selesai kita bisa lanjut untuk menggunakan `foundry` untuk menyelesaikan ethernaut ini dan belajar cara melakukan exploit smart contract dari bad practice nya developer.

Oke begitu saja post level 0 ini, sampai berjumpa di level berikutnya yang lebih menantang karena kita akan melakukan code review 😦

Seeya 👋🐢
