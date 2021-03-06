---
title: Unidades de Tempo
actions: ['verificarResposta', 'dicas']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiefactory.sol": |
        pragma solidity ^0.4.19;

        import "./ownable.sol";

        contract ZombieFactory is Ownable {

            event NewZombie(uint zombieId, string name, uint dna);

            uint dnaDigits = 16;
            uint dnaModulus = 10 ** dnaDigits;
            // 1. Defina o `cooldownTime` aqui

            struct Zombie {
                string name;
                uint dna;
                uint32 level;
                uint32 readyTime;
            }

            Zombie[] public zombies;

            mapping (uint => address) public zombieToOwner;
            mapping (address => uint) ownerZombieCount;

            function _createZombie(string _name, uint _dna) internal {
                // 2. Atualize a seguinte linha:
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
                randDna = randDna - randDna % 100;
                _createZombie(_name, randDna);
            }

        }
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefactory.sol";

        contract KittyInterface {
          function getKitty(uint256 _id) external view returns (
            bool isGestating,
            bool isReady,
            uint256 cooldownIndex,
            uint256 nextActionAt,
            uint256 siringWithId,
            uint256 birthTime,
            uint256 matronId,
            uint256 sireId,
            uint256 generation,
            uint256 genes
          );
        }

        contract ZombieFeeding is ZombieFactory {

          KittyInterface kittyContract;

          function setKittyContractAddress(address _address) external onlyOwner {
            kittyContract = KittyInterface(_address);
          }

          function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            if (keccak256(_species) == keccak256("kitty")) {
              newDna = newDna - newDna % 100 + 99;
            }
            _createZombie("NoName", newDna);
          }

          function feedOnKitty(uint _zombieId, uint _kittyId) public {
            uint kittyDna;
            (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
            feedAndMultiply(_zombieId, kittyDna, "kitty");
          }

        }
      "ownable.sol": |
        /**
         * @title Ownable
         * @dev The Ownable contract has an owner address, and provides basic authorization control
         * functions, this simplifies the implementation of "user permissions".
         */
        contract Ownable {
          address public owner;

          event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

          /**
           * @dev The Ownable constructor sets the original `owner` of the contract to the sender
           * account.
           */
          function Ownable() public {
            owner = msg.sender;
          }

          /**
           * @dev Throws if called by any account other than the owner.
           */
          modifier onlyOwner() {
            require(msg.sender == owner);
            _;
          }

          /**
           * @dev Allows the current owner to transfer control of the contract to a newOwner.
           * @param newOwner The address to transfer ownership to.
           */
          function transferOwnership(address newOwner) public onlyOwner {
            require(newOwner != address(0));
            OwnershipTransferred(owner, newOwner);
            owner = newOwner;
          }

        }
    answer: >
      pragma solidity ^0.4.19;

      import "./ownable.sol";

      contract ZombieFactory is Ownable {

          event NewZombie(uint zombieId, string name, uint dna);

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;
          uint cooldownTime = 1 days;

          struct Zombie {
            string name;
            uint dna;
            uint32 level;
            uint32 readyTime;
          }

          Zombie[] public zombies;

          mapping (uint => address) public zombieToOwner;
          mapping (address => uint) ownerZombieCount;

          function _createZombie(string _name, uint _dna) internal {
              uint id = zombies.push(Zombie(_name, _dna, 1, uint32(now + cooldownTime))) - 1;
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
              randDna = randDna - randDna % 100;
              _createZombie(_name, randDna);
          }

      }
---

A propriedade `level` (n??vel) ?? bem auto explicativa. Mais tarde, quando criarmos um sistema de batalha, zumbis que vencerem mais batalhas ir??o subir de n??vel com o tempo e ganhar acesso a novas habilidades.

A propriedade `readyTime` requer um pouco mais de explica????o. A ideia ?? adicionar um "tempo de esfriamento", uma quantidade de tempo que o zumbi precisa esperar para se alimentar ou atacar antes que possar alimentar / atacar novamente. Sem isto, o zumbi poderia atacar m??ltiplas 1.000 vezes por dia, algo que poderia tornar o jogo muito f??cil.

Para manter o registro de quanto tempo um zumbi precisa esperar at?? o poder atacar novamente, podemos usar as unidades tempo em Solidity.

## Time units
## Unidades de tempo

Solidity fornece algumas unidades nativas para lidar com o tempo.

A vari??vel `now` (agora) ir?? retornar o *unix timestamp* atual (o n??mero de segundos que passou desde 1 de Janeiro de 1970). O tempo unix no momento que escrevo este tutorial ?? `1515527488`.

>Nota: O tempo unix ?? tradicionalmente guardado em um n??mero de 32-bits. Isto ir?? causar o problema do "Ano 2038", quando os unix timestamps ir??o sobre carregar e quebrar um monte de sistema legados. Ent??o se queremos a nossa DApp continuar rodando mais do que 20 anos a partir de agora, podemos usar um n??mero de 64-bits - mas nossos usu??rios ter??o que gastar mais gas para usar a nossa DApp no momento. Decis??es de projeto!

Solidity tamb??m contem os tempo em unidades de `seconds` (segundos), `minutes` (minutos), `hours` (horas), `days` (dias), `weeks` (semanas). Estes ir??o converter para um `uint` do n??mero em segundos no per??odo de tempo. Ent??o `1 minutes` s??o `60`, `1 hours` s??o `3600`(60 segundos x 60 minutos), `1 days` s??o `86400` (24 hours x 60 minutos x 60 segundos), etc.

Segue um exemplo de como essas unidades de tempo s??o ??teis:

```
uint lastUpdated;

// Atribui `lastUpdated` para `now`
function updateTimestamp() public {
  lastUpdated = now;
}

// Ir?? retornar `true` se 5 minutos se passaram desde de que `updateTimestamp` foi
// chamado, `false` se os 5 minutos ainda passaram
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```

Podemos usar estas unidades de tempo para o `cooldown` (esfriar) do Zumbi.

## Vamos testar

Vamos adicionar o tempo de esfriamento em nossa DApp, e ao fazer isso os zumbis ter??o que esperar **1 dia** ap??s atacar ou se alimentar para atacar novamente.

1. Declare um `uint` chamado `cooldownTime`, e atribua-lhe o valor igual a `1 days`. (N??o ligue para p??ssima gram??tica - se voc?? atribuir igual a "1 day", o mesmo n??o ir?? compilar!)

2. Uma vez que adicionamos o `level` e `readyTime` a nossa estrutura `Zombie` no cap??tulo anterior, precisamos atualizar a fun????o `_createZombie()` com o n??mero correto de argumentos ao criar uma nova estrutura `Zombie`

  Atualize a linha `zombies.push` e adicione 2 argumentos a mais: `1` (para `level`), e `uint32(now + cooldownTime)` (para `readyTime`).

>Nota: O `uint32(...)` ?? necess??rio porque `now` (agora) returna um `uint256` por padr??o. Ent??o precisamos deixar expl??cito a convers??o para um `uint32`.

`now + cooldownTime` ser?? igual o *unix timestamp* atual (em segundos) mais o n??mero de segundos em 1 dia - que ?? igual o n??mero de segundos de 1 dia a partir de agora. Mais tarde iremos comparar para ver se este `readyTime` do zumbi ?? maior do que `now` (agora) e ver se passou tempo o suficiente para usar o zumbi novamente.

Implementaremos a funcionalidade para limitar as a????es baseado no `readyTime` no pr??ximo cap??tulo.
