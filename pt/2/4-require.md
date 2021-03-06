---
title: Requerer (Require)
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
              zombieToOwner[id] = msg.sender;
              ownerZombieCount[msg.sender]++;
              NewZombie(id, _name, _dna);
          }

          function _generateRandomDna(string _str) private view returns (uint) {
              uint rand = uint(keccak256(_str));
              return rand % dnaModulus;
          }

          function createRandomZombie(string _name) public {
              // comece aqui
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
              require(ownerZombieCount[msg.sender] == 0);
              uint randDna = _generateRandomDna(_name);
              _createZombie(_name, randDna);
          }

      }
---

Na li????o 1, possibilitamos os usu??rios de criar novos zumbis chamando a fun????o `createRandomZombie` e colocando um nome. Por??m, se os usu??rios continuarem chamando esta fun????o e de forma ilimitada criando zumbis em seus ex??rcitos, o jogo n??o teria tanta gra??a.

Vamos fazer assim, cada jogador s?? pode chamar esta fun????o uma vez. Desta maneira novos jogadores ir??o chamar s?? quando come??arem o jogo pela primeira vez, para criar o primeiro zumbi do ex??rcito.

Como podemos fazer esta fun????o ser chamada somente uma vez por jogador?

Para isso n??s vamos usar o `require` (requerer). `require` faz com que a fun????o lance um erro e pare a execu????o se alguma condi????o n??o for verdadeira:

```
function sayHiToVitalik(string _name) public returns (string) {
  // Compara se _name ?? igual ?? "Vitalik". Lan??a um erro e termina se n??o for verdade.
  // (Lembrete: Solidity n??o tem uma forma nativa de comparar strings, ent??o
  // temos que comparar os hashes keccak256 para verificar a igualdade)
  require(keccak256(_name) == keccak256("Vitalik"));

  // Se ?? verdade, prossiga com a fun????o:
  return "Ol??!";
}
```

Se voc?? chamar esta fun????o com `sayHiToVitalik("Vitalik")`, ela ir?? retornar "Ol??!". Se voc?? chamar esta fun????o com outra entrada, ela ir?? lan??ar um erro e n??o ir?? executar.

Sendo assim, `require` ?? muito ??til para verificar certas condi????es que devem ser verdadeiras antes de executar uma fun????o.

# Vamos testar

Em nosso jogo de zumbi, n??s n??o queremos que o usu??rio possa criar zumbis ilimitadamente em seus ex??rcitos ao chamar a fun????o `createRandomZombie` consecutivamente - acabaria com a gra??a do jogo.

Vamos usar o `require` para ter certeza que esta fun????o s?? ser?? executada uma vez por usu??rio, quando precisarem criar o primeiro zumbi.

1. Coloque uma declara????o de `require` no come??o da fun????o `createRandomZombie`. A fun????o deve checar para ter certeza que `ownerZombieCount[msg.sender]` ?? igual a `0`, e lan??ar um erro caso o contr??rio.

> Nota: Em Solidity, n??o importa qual o termo voc?? usar primeiro - ambas as formas funcionam. Por??m, desde que o nosso checador de resposta ?? bem b??sico, ele s?? aceita um tipo de resposta correta - ele espera que `ownerZombieCount[msg.sender]` esteja em primeiro.
