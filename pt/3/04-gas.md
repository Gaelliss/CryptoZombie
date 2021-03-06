---
title: Gas
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

            struct Zombie {
                string name;
                uint dna;
                // Adicione o novo dado aqui
            }

            Zombie[] public zombies;

            mapping (uint => address) public zombieToOwner;
            mapping (address => uint) ownerZombieCount;

            function _createZombie(string _name, uint _dna) internal {
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
---

??timo! Agora sabemos como atualizar por????es importantes da DApp enquanto prevenimos que outros usu??rios estraguem com nossos contratos.

Vamos ver outra maneira que Solidity ?? um tanto diferente de outras linguagens de programa????o:

## Gas ?????o combust??vel utilizado por DApps Ethereum

Em Solidity, seus usu??rios tem que pagar toda vez que executam uma fun????o em sua DApp usando uma moeda chamada **_gas_**. Usu??rios compram gas com Ether (a moeda no Ethereum), ent??o os seus usu??rios precisam gastar ETH para executar fun????es em sua DApp.

Quanto gas ?? preciso para executar uma fun????o depende o qu??o complexo ?? a l??gica desta fun????o. Cada opera????o individual tem um **_custo em gas_** baseado mais ou menos em quanto recursos computacionais ser??o necess??rios para realizar essa opera????o (exemplo: escrever em storage ?? muito mais caro do que adicionar dois inteiros). O total de **_custo em gas_** da sua fun????o ?? soma de todos os custo de todas as opera????es de forma individuais.

E porque executar fun????es custam dinheiro real para os seus usu??rios, otimiza????o do c??digo ?? muito mais importante em Ethereum do que em qualquer outra linguagem de programa????o. Se o seu c??digo ?? desleixado, seus usu??rios ter??o que pagar muito mais para executar suas fun????es - e isto pode adicionar milh??es de d??lares em custos desnecess??rios atrav??s de milhares de usu??rios.

## Por que o gas ?? necess??rio?

Ethereum ?? como um grande, lento, mas extremamente seguro computador. Quando voc?? executa uma fun????o, cada um dos n??s na rede precisa rodar esta mesma fun????o e verificar suas sa??das - milhares de nos verificando cada execu????o de cada fun????o ?? o que faz do Ethereum decentralizado, e seus dados imut??veis e resistentes a censura.

Os criadores do Ethereum queriam ter certeza que ningu??m poderia entupir a rede com la??os infinitos, ou estragar todos os recursos da rede com computa????es realmente intensivas. Ent??o eles fizeram com que as transa????es n??o fossem gr??tis, e os usu??rios tem que pagar pelo tempo de computa????o como tamb??m pela guarda dos dados

> Nota: Isto n??o ?? necessariamente verdade em sidechains, como a que os autores do CryptoZombies est??o construindo na Loom Network. E provavelmente nunca ir?? fazer sentido rodar um jogo como World of Warcraft diretamente na rede mainnet do Ethereum - o custo em gas seria proibitivamente caro. Mas este pode rodar em uma sidechain com um algor??timo de consenso diferente. Iremos falar mais sobre os tipos de DApps que voc?? poderia implantar em sidechains vs a mainnet do Ethereum em futuras li????es.

## Empacotamento da estrutura para economizar gas

Na Li????o 1, n??s mencionamos que existem outros tipos de `uint`s: `uint8`, `uint16`, `uint32`, etc.

Normalmente n??o h?? benef??cio algum em usar estes subtipos porque Solidity reserva 256 bits de espa??o independente do tamanho do `uint`. Por exemplo, usar `uint8` ao inv??s de `uint` (`uint256`) n??o ir?? economizar gas algum.

Mas h?? uma exce????o para isto: dentro das `struct`s.

Se voc?? tiver m??ltiplas `uint`s dentro de uma `struct`, usando um tamanho menor de `uint` quando poss??vel ir?? permitir o Solidity de empacotar essas vari??veis juntas para usar menos espa??o. Por exemplo:

```
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` ir?? custar menos gas que `normal` porque usar o empacotamento
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30);
```

Por essa raz??o, dentro de uma estrutura voc?? ir?? querer usar o menor subtipo de integer que voc?? puder. Voc?? tamb??m quer juntar tipos de dados id??nticos (exemplo: colando um perto do outro dentro da estrutura) ent??o Solidity pode minimizar o espa??o requerido para guardar a estrutura. Por exemplo, a estrutura com campos `uint c; uint32 a; uint32 b;` ir?? custar menos gas que a estrutura com os campos `uint32 a; uint c; uint32 b;` porque o  os campos `uint32` est??o agrupados.

## Vamos testar

Nesta li????o, iremos adicionar 2 novas caracter??sticas para os nossos zumbis: `level` (n??vel) e `readyTime` - que mais tarde ser??o utilizados parar implementar o tempo de *cooldown* (esfriar) para limitar o qu??o frequente um zumbi pode se alimentar.

Vamos voltar ent??o para `zombiefactory.sol`.

1. Adicione duas propriedades para a nossa estrutura `Zombie`: `level` (um `uint32`), e `readyTime` (tamb??m um `uint32`). Queremos empacotar esses tipos juntos, ent??o vamos coloc??-los no final da estrutura.

Um 32 bits ?? mais do que o suficiente para guardar o n??vel do zumbi e uma marca de hor??rio, isso ir?? economizar-nos alguns custos em gas por empacotar o dado mais junto do que usar um `uint` (256-bits).
