---
title: DNA Zumbi
actions: ['verificarResposta', 'dicas']
material:
  editor:
    language: sol
    startingCode:
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefactory.sol";

        contract ZombieFeeding is ZombieFactory {

          function feedAndMultiply(uint _zombieId, uint _targetDna) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            // comece aqui
          }

        }
      "zombiefactory.sol": |
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
    answer: >
      pragma solidity ^0.4.19;

      import "./zombiefactory.sol";

      contract ZombieFeeding is ZombieFactory {

        function feedAndMultiply(uint _zombieId, uint _targetDna) public {
          require(msg.sender == zombieToOwner[_zombieId]);
          Zombie storage myZombie = zombies[_zombieId];
          _targetDna = _targetDna % dnaModulus;
          uint newDna = (myZombie.dna + _targetDna) / 2;
          _createZombie("NoName", newDna);
        }

      }
---

Vamos terminar de escrever a fun????o `feedAndMultiply`.

A f??rmula para o c??lculo do novo DNA zumbi ?? simples: Simplesmente a m??dia entre o DNA do zumbi alimentado e o DNA do alvo.

Por exemplo:

```
function testDnaSplicing() public {
  uint zombieDna = 2222222222222222;
  uint targetDna = 4444444444444444;
  uint newZombieDna = (zombieDna + targetDna) / 2;
  // ^ ser?? igual a 3333333333333333
}
```

Mais tarde podemos deixar a nossa f??rmula mais complicada se quisermos, como adicionar alguma aleatoriedade ao DNA do novo zumbi. Mas por enquanto vamos mante-l?? simples - n??s sempre podemos voltar aqui mais tarde.

# Vamos testar

1. Primeiro precisamos ter certeza que o `_targetDna` n??o ?? maior do que 16 d??gitos. Para isso n??s vamos definir `_targetDna` igual ?? `_targetDna % dnaModulus` para somente ter os ??ltimos 16 d??gitos.

2. Depois nossa fun????o deve declarar um `uint` chamado `newDna`, e definir igual a m??dia do DNA do `myZombie` e `_targetDna` (como o exemplo acima).

  > Nota: Voc?? pode acessar as propriedades do `myZombie` usando `myZombie.name` e `myZombie.dna`

3. Uma vez que temos o novo DNA, vamos chamar a fun????o `_createZombie`. Voc?? pode olhar na aba `zombiefactory.sol` se voc?? esqueceu quais par??metros esta fun????o precisa. Perceba que ela requer um nome, ent??o vamos definir o nome do nosso novo zumbi como `"NoName"` por enquanto - n??s podemos escrever uma fun????o parar mudar os nomes mais tarde.

> Nota: Sobre as "sujeiras" no c??digo Solidity, talvez voc?? tenha notado algum problema em nosso c??digo aqui! N??o se preocupe, vamos arrumar isso no pr??ximo cap??tulo ;)
