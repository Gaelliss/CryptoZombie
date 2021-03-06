---
title: O Que Zumbis Comem?
actions: ['verificarResposta', 'dicas']
material:
  editor:
    language: sol
    startingCode:
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefactory.sol";

        // Crie a KittyInterface aqui

        contract ZombieFeeding is ZombieFactory {

          function feedAndMultiply(uint _zombieId, uint _targetDna) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            _createZombie("NoName", newDna);
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
                _createZombie(_name, randDna);
            }

        }
    answer: >
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

        function feedAndMultiply(uint _zombieId, uint _targetDna) public {
          require(msg.sender == zombieToOwner[_zombieId]);
          Zombie storage myZombie = zombies[_zombieId];
          _targetDna = _targetDna % dnaModulus;
          uint newDna = (myZombie.dna + _targetDna) / 2;
          _createZombie("NoName", newDna);
        }

      }
---

Esta na hora de alimentar os nossos zumbis! E o que zumbis mais gostam de comer?

Bom, acontece que CryptoZombies adoram comer...

**CryptoKitties!** ????????????

(Sim, ?? s??rio ????)

Para fazer isso n??s precisamos ler o kittyDna do smart contract do CryptoKitties. Podemos fazer isso porque o dado dos CryptoKitties ?? gravado de forma aberta na blockchain. O blockchain n??o ?? legal?!

N??o se preocupe - nosso jogo atual n??o ir?? machucar qualquer CryptoKitty. N??s somente vamos *ler* os dados do CryptoKitties, n??s n??o podemos delet??-lo ????

## Interagindo com outros contratos

Para que o nosso contrato converse com outro contrato na blockchain que n??o ?? nosso, primeiro temos que definir uma **_interface_**.

Vamos ver um simples exemplo. Digamos que existe um contrato na blockchain que se parece com isto:

```
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

Este seria um simples contrato onde qualquer um pode guardar um n??mero da sorte, e esse n??mero seria associado ao seu endere??o no Ethereum. Ent??o qualquer um pode olhar o n??mero desta pessoa somente passando o endere??o dessa pessoa.

Agora digamos que n??s temos um contrato externo que quer ler o dado deste contrato usando a fun????o `getNum`.

Primeiro n??s gostar??amos de definir uma **_interface_** para o contrato `LuckyNumber`:

```
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

Perceba que isto parece com a defini????o de um contrato, com poucas diferen??as. Primeiro, declaramos somente as fun????es que queremos interagir - neste caso `getNum` - e n??s n??o mencionamos qualquer outra fun????o ou vari??veis de estado.

Segundo, n??s n??o definimos os corpos das fun????es. Ao inv??s dos "braces" (`{` e `}`), n??s simplesmente terminamos declara????o da fun????o com um ponto-e-v??rgula (`;`).

Ent??o isso se parece com um esqueleto de contrato. Isto ?? como o compilador sabe sobre uma interface.

Ao incluir esta interface no c??digo da nossa aplica????o distribu??da (dapp) nosso contrato sabe como as fun????es do outro contrato se parecem, e como execut??-las, e qual tipo de resposta esperar.

Iremos na verdade executar fun????es em um outro contrato em nossa pr??xima li????o, mas por enquanto vamos declarar as nossas interfaces para o contrato do CryptoKitties.

# Vamos testar

N??s j?? olhamos o c??digo fonte do CryptoKitties pra voc??, e encontramos a fun????o chamada `getKitty` que retorna todos os dados do gato, incluindo seus "genes" (que ?? o que nosso jogo de zumbi precisa para formar um novo zumbi!).

A fun????o parece com isso:

```
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
) {
    Kitty storage kit = kitties[_id];

    // se esta vari??vel for 0 ent??o n??o ?? gestante
    isGestating = (kit.siringWithId != 0);
    isReady = (kit.cooldownEndBlock <= block.number);
    cooldownIndex = uint256(kit.cooldownIndex);
    nextActionAt = uint256(kit.cooldownEndBlock);
    siringWithId = uint256(kit.siringWithId);
    birthTime = uint256(kit.birthTime);
    matronId = uint256(kit.matronId);
    sireId = uint256(kit.sireId);
    generation = uint256(kit.generation);
    genes = kit.genes;
}
```

A fun????o parece um pouco diferente da fun????o que n??s precisamos. Voc?? pode ver o que ela retorna... um monte de valores diferentes. Se voc?? vem de uma linguagem de programa????o como Javascript, isto ?? diferente - em Solidity voc?? pode retornar mais de um valor por fun????o.

Agora que n??s sabemos como a fun????o deve ser, podemos us??-la para criar uma interface:

1. Defina uma interface chamada `KittyInterface`. Lembrando, isto parece com a cria????o um novo contrato - usamos a palavra reservada `contract`.

2. Dentro da interface, defina a fun????o `getKitty` (que deve ser uma c??pia da fun????o acima, mas com o ponto-e-v??rgula ap??s a declara????o `returns`, ao inv??s de tudo que esta dentro dos braces `{` e `}`).
