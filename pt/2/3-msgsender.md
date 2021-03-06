---
title: Msg.sender
actions: ['verificarResposta', 'dicas']
material:
  editor:
    language: sol
    startingCode: |
      pragma solidity ^0.4.19;

      contract ZombieFactory {

          event NewZombie(uint zombieId, string name, uint dna);

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;

          struct Zombie {
              string name;
              uint dna;
          }

          Zombie[] public zombies;

          mapping (uint => address) public zombieToOwner;
          mapping (address => uint) ownerZombieCount;

          function _createZombie(string _name, uint _dna) private {
              uint id = zombies.push(Zombie(_name, _dna)) - 1;
              // comece aqui
              NewZombie(id, _name, _dna);
          }

          function _generateRandomDna(string _str) private view returns (uint) {
              uint rand = uint(keccak256(_str));
              return rand % dnaModulus;
          }

          function createRandomZombie(string _name) public {
              uint randDna = _generateRandomDna(_name);
              _createZombie(_name, randDna);
          }

      }
    answer: >
      pragma solidity ^0.4.19;


      contract ZombieFactory {

          event NewZombie(uint zombieId, string name, uint dna);

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;

          struct Zombie {
              string name;
              uint dna;
          }

          Zombie[] public zombies;

          mapping (uint => address) public zombieToOwner;
          mapping (address => uint) ownerZombieCount;

          function _createZombie(string _name, uint _dna) private {
              uint id = zombies.push(Zombie(_name, _dna)) - 1;
              zombieToOwner[id] = msg.sender;
              ownerZombieCount[msg.sender]++;
              NewZombie(id, _name, _dna);
          }

          function _generateRandomDna(string _str) private view returns (uint) {
              uint rand = uint(keccak256(_str));
              return rand % dnaModulus;
          }

          function createRandomZombie(string _name) public {
              uint randDna = _generateRandomDna(_name);
              _createZombie(_name, randDna);
          }

      }
---

Agora temos nossos mapeamentos para guardar os registros de quem ?? dono de cada zumbi, queremos atualizar o m??todo `_createZombie` para us??-los.

E para fazer isto, precisamos usar algo chamado `msg.sender`.

## msg.sender

Em Solidity, existem certas vari??veis globais que est??o dispon??veis em todas fun????es. Umas dessas ?? a `msg.sender`, que refere-se ao `address` (endere??o) da pessoa (ou smart contract) que chamou a fun????o em atual.

> Nota: Em Solidity, uma fun????o sempre precisa iniciar como uma chamada externa. Um contrato ir?? somente descansar no blockchain sem fazer nada at?? que algu??m chame alguma das suas fun????es. Ent??o sempre haver?? um `msg.sender`.

Segue um exemplo de uso do `msg.sender` e atualiza????o de um `mapping`:

```
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // Atualiza o nosso mapeamento `favoriteNumber` para guardar o `_myNumber` utilizando `msg.sender`
  favoriteNumber[msg.sender] = _myNumber;
  // ^ A sintaxe para guardar o dado em um mapeamento ?? parecida com a dos arrays
}

function whatIsMyNumber() public view returns (uint) {
  // Recupera o valor guardado em um endere??o de quem transmitiu (sender)
  // Ser?? `0` se o transmissor (sender) n??o chamou a fun????o `setMyNumber` ainda
  return favoriteNumber[msg.sender];
}
```

Neste exemplo trivial, qualquer um pode chamar a fun????o `setMyNumber` e guardar um `uint` em nosso contrato, que ser?? amarrado ao seu endere??o. Ent??o quando eles chamarem a fun????o `whatIsMyNumber`, seria retornado o `uint` que eles guardaram.

Usar o `msg.sender` fornece ?? voc?? a seguran??a do blockchain do Ethereum - a ??nica maneira de algu??m modificar o dado de outra pessoas seria roubando a chave privada associada ao endere??o no Ethereum.

# Vamos testar

Vamos atualizar o nosso m??todo `_createZombie` da li????o 1 para atribuir a propriedade do zumbi a pessoa que executou a fun????o.

1. Primeiro, ap??s termos o novo `id` do zumbi, vamos atualizar o nosso mapeamento `zombieToOwner` para guardar o `msg.sender` sob o `id`.

2. Segundo, vamos incrementar o contador `ownerZombieCount` para este `msg.sender`

Em Solidity, voc?? pode incrementar um `uint` com `++`, como em javascript:

```
uint number = 0;
number++;
// `number` agora ?? `1`
```

A sua resposta final para este cap??tulo deve ter 2 linhas de c??digo.
