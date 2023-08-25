---
title: "Level 2 - Fallout"
date: 2023-08-25T22:22:43+07:00
draft: false
tags: ["Ethernaut"]
categories: ["Smart Contract Securiy"]
---

Haloo kita lanjutkan kembali series ethernaut ini di Level 2 yaitu Fallout, Lesgo!

Pada level kali ini kita diberikan source code dari smart contractnya sebagai berikut

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Fallout {

  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;

  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```

> Goals : `player` harus claim `ownership` contract tersebut

### Breakdown the code

Oke mari kita breakdown smart contract yang bisa kita exploit

```solidity
pragma solidity ^0.6.0; **(1)**

import 'openzeppelin-contracts-06/math/SafeMath.sol'; **(2)**

contract Fallout {

  using SafeMath for uint256; **(3)**
  mapping (address => uint) allocations; **(4)**
  address payable public owner; **(5)**

  /* constructor */
  function Fal1out() public payable { **(6)**
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

<snip>
```

1. Smart contract ini menggunakan compiler solidity v0.6.0 ke atas
2. Melakukan import library `SafeMath.sol`
3. Deklarasi penggunaan `SafeMath` dengan tipe data uint256
4. Deklarasi variable `allocations` bertipe data mapping yang akan menyimpan alokasi uint
5. Deklarasi variable `owner` yang bertipe data `address` dan `payable` sehingga address ini dapat mengirim dan menerima ether.
6. Constructor dari contract `Fallout` yang akan menassign `msg.sender` menjadi `owner`

### Explainoit (Explain & Exploit)

Baik setelah kita memahami code yang sudah dijelaskan secara singkat diatas, jika kita perhatikan yang menarik perhatian pertama kali adalah `constructor`nya berbeda dengan smart contract sebelumnya. Kenapa bisa?

Jadi constructor pada [Solidity v0.4.21](https://docs.soliditylang.org/en/v0.4.21/contracts.html) kebawah mempunyai sebuah constructor seperti berikut

```solidity
pragma solidity ^0.4.11;

contract A {
    uint public a;

    function A(uint _a) internal {
        a = _a;
    }
}
```

Jika kita perhatikan constructornya itu akan memanggil nama contractnya, sedangkan di versi 0.4.22 ke atas constructornya seperti yang kita sudah tahu sebelumnya

```solidity
constructor(){
	owner = msg.sender;
}
```

Nah pada smart contract ini menggunakan Solidity v0.6.0 yang sangat aneh jika constructornya seperti source code yang dikasih. Tetapi jika kalian lihat nama constructornya pada source code ini sedikit aneh karena penamaan `Fallout` nya itu salah satu hurufnya menggunakan angka `1` sehingga menjadi `Fal1out`

Dan jika kita coba ubah nama constructornya menjadi `Fallout` sama seperti nama contract akan terdapat error seperti berikut

{{< figure align=center src="/images/20230825/notAllowed.png" title="Not allowed same as contract" >}}

Ini menandakan bahwa v0.4.22 ke atas tidak bisa memiliki nama _function_ yang sama seperti nama contract. Jika kita coba jalankan perintah berikut

```jsx
await contract.owner();
```

Ternyata ownernya tidak di define, ini karena `Fal1out` ini merupakan sebuah public function sehingga tidak ada constructor yang berjalan

{{< figure align=center src="/images/20230825/contractowner.png" title="Fallout Contract Owner" >}}

Oleh karena itu kita bisa akses function `Fallout()` langsung untuk mengubah ownershipnya dengan menjalankan perintah berikut

```jsx
await contract.Fal1out();
```

{{< figure align=center src="/images/20230825/changeowner.png" title="Change Owner" >}}

Dan kita cek kembali ownernya dengan menjalankan perintah yang sama

```jsx
await contract.owner();
```

Ownernya sudah berubah menjadi address kita/`player`

{{< figure align=center src="/images/20230825/changed.png" title="Owner changed" >}}

Level ini memiliki [SWC-118: Incorrect Constructor Name](https://swcregistry.io/docs/SWC-118/) dan level ini juga mempunyai case real world pada perusahaan yang bernama `Dynamic Pyramid` berubah nama menjadi `Rubixi` , tetapi mereka lupa untuk mengubah nama constructor methodnya seperti berikut

```solidity
contract Rubixi {
  address private owner;
  function DynamicPyramid() { owner = msg.sender; }
  function collectAllFees() { owner.transfer(this.balance) }
  ...
```

Sehingga attacker dapat memanggil old constructor untuk claim ownership contractnya

Okey segitu saja untuk post kali ini, kita akan bertemu lagi di level berikutnya

Seeya 👋🐢
