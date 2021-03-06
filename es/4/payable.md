---
title: Payable
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiehelper.sol": |
        pragma solidity ^0.4.25;

        import "./zombiefeeding.sol";

        contract ZombieHelper is ZombieFeeding {

          // 1. Declare levelUpFee aqui

          modifier aboveLevel(uint _level, uint _zombieId) {
            require(zombies[_zombieId].level >= _level);
            _;
          }

          // 2. Escriba la funcionlevelUp aqui

          function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
            require(msg.sender == zombieToOwner[_zombieId]);
            zombies[_zombieId].name = _newName;
          }

          function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
            require(msg.sender == zombieToOwner[_zombieId]);
            zombies[_zombieId].dna = _newDna;
          }

          function getZombiesByOwner(address _owner) external view returns(uint[]) {
            uint[] memory result = new uint[](ownerZombieCount[_owner]);
            uint counter = 0;
            for (uint i = 0; i < zombies.length; i++) {
              if (zombieToOwner[i] == _owner) {
                result[counter] = i;
                counter++;
              }
            }
            return result;
          }

        }
      "zombiefeeding.sol": |
        pragma solidity ^0.4.25;

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

          function _triggerCooldown(Zombie storage _zombie) internal {
            _zombie.readyTime = uint32(now + cooldownTime);
          }

          function _isReady(Zombie storage _zombie) internal view returns (bool) {
              return (_zombie.readyTime <= now);
          }

          function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            require(_isReady(myZombie));
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
              newDna = newDna - newDna % 100 + 99;
            }
            _createZombie("NoName", newDna);
            _triggerCooldown(myZombie);
          }

          function feedOnKitty(uint _zombieId, uint _kittyId) public {
            uint kittyDna;
            (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
            feedAndMultiply(_zombieId, kittyDna, "kitty");
          }
        }
      "zombiefactory.sol": |
        pragma solidity ^0.4.25;

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
                emit NewZombie(id, _name, _dna);
            }

            function _generateRandomDna(string _str) private view returns (uint) {
                uint rand = uint(keccak256(abi.encodePacked(_str)));
                return rand % dnaModulus;
            }

            function createRandomZombie(string _name) public {
                require(ownerZombieCount[msg.sender] == 0);
                uint randDna = _generateRandomDna(_name);
                randDna = randDna - randDna % 100;
                _createZombie(_name, randDna);
            }

        }
      "ownable.sol": |
       pragma solidity ^0.4.25;

        /**
        * @title Ownable
        * @dev The Ownable contract has an owner address, and provides basic authorization control
        * functions, this simplifies the implementation of "user permissions".
        */
        contract Ownable {
          address private _owner;

          event OwnershipTransferred(
            address indexed previousOwner,
            address indexed newOwner
          );

          /**
          * @dev The Ownable constructor sets the original `owner` of the contract to the sender
          * account.
          */
          constructor() internal {
            _owner = msg.sender;
            emit OwnershipTransferred(address(0), _owner);
          }

          /**
          * @return the address of the owner.
          */
          function owner() public view returns(address) {
            return _owner;
          }

          /**
          * @dev Throws if called by any account other than the owner.
          */
          modifier onlyOwner() {
            require(isOwner());
            _;
          }

          /**
          * @return true if `msg.sender` is the owner of the contract.
          */
          function isOwner() public view returns(bool) {
            return msg.sender == _owner;
          }

          /**
          * @dev Allows the current owner to relinquish control of the contract.
          * @notice Renouncing to ownership will leave the contract without an owner.
          * It will not be possible to call the functions with the `onlyOwner`
          * modifier anymore.
          */
          function renounceOwnership() public onlyOwner {
            emit OwnershipTransferred(_owner, address(0));
            _owner = address(0);
          }

          /**
          * @dev Allows the current owner to transfer control of the contract to a newOwner.
          * @param newOwner The address to transfer ownership to.
          */
          function transferOwnership(address newOwner) public onlyOwner {
            _transferOwnership(newOwner);
          }

          /**
          * @dev Transfers control of the contract to a newOwner.
          * @param newOwner The address to transfer ownership to.
          */
          function _transferOwnership(address newOwner) internal {
            require(newOwner != address(0));
            emit OwnershipTransferred(_owner, newOwner);
            _owner = newOwner;
          }
        }
    answer: >
      pragma solidity ^0.4.25;

      import "./zombiefeeding.sol";

      contract ZombieHelper is ZombieFeeding {

        uint levelUpFee = 0.001 ether;

        modifier aboveLevel(uint _level, uint _zombieId) {
          require(zombies[_zombieId].level >= _level);
          _;
        }

        function levelUp(uint _zombieId) external payable {
          require(msg.value == levelUpFee);
          zombies[_zombieId].level++;
        }

        function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
          require(msg.sender == zombieToOwner[_zombieId]);
          zombies[_zombieId].name = _newName;
        }

        function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
          require(msg.sender == zombieToOwner[_zombieId]);
          zombies[_zombieId].dna = _newDna;
        }

        function getZombiesByOwner(address _owner) external view returns(uint[]) {
          uint[] memory result = new uint[](ownerZombieCount[_owner]);
          uint counter = 0;
          for (uint i = 0; i < zombies.length; i++) {
            if (zombieToOwner[i] == _owner) {
              result[counter] = i;
              counter++;
            }
          }
          return result;
        }

      }
---

Hasta ahora, hemos cubierto unos cuantos **_modificadores de funci??n_**. Puede resultar dif??cil tratar de recordar todo, as?? que hagamos un breve repaso:

1. Tenemos modificadores de visibilidad que controlan desde d??nde y cu??ndo la funci??n puede ser llamada: `private` significa que s??lo puede ser llamada desde otras funciones dentro del contrato; `internal` es como `private` pero tambi??n puede ser llamada por contratos que hereden desde este; `external` s??lo puede ser llamada desde afuera del contrato; y finalmente `public` puede ser llamada desde cualquier lugar, tanto internamente como externamente.

2. Tambi??n tenemos modificadores, los cuales nos dicen c??mo interact??a la funci??n con la BlockChain: `view` nos indica que al ejecutar la funci??n, ning??n dato ser?? guardado/cambiado. `pure` nos indica que la funci??n no s??lo no guarda ning??n dato en la blockchain, si no que tampoco lee ning??n dato de la blockchain. Ambos no cuestan nada de combustible para llamar si son llamados externamente desde afuera del contrato (pero si cuestan combustible si son llamado internamente por otra funci??n).

3. Luego tenemos los `modifiers` personalizados, de los cuales aprendimos en la Lecci??n 3: `onlyOwner` y `aboveLevel`, por ejemplo. Para estos podemos definir la l??gica personalizada para determinar c??mo afectan a una funci??n.

Todos estos modificadores pueden ser apilados juntos en una definici??n de funci??n de la siguiente manera:

```
function test() external view onlyOwner anotherModifier { /* ... */ }
```

En este cap??tulo, vamos a presentar un modificador de funci??n m??s: `payable`.

## El Modificador `payable`

Las funciones `payable` son parte de lo que hace de Solidity y Ethereum algo tan genial ??? son un tipo de funci??n especial que pueden recibir Ether.

Pienselo por un momento. Cuando llama una funci??n API en un servidor web normal, no puede enviar d??lares (USD$) junto con su llamada de funci??n ??? ni enviar Bitcoin.

Pero en Ethereum, ya que tanto el dinero (_Ether_), los datos (*payload de transacci??n*) y el mismo c??digo de contrato viven en Ethereum, es posible para usted llamar a una funci??n **y** pagar dinero por el contrato al mismo tiempo.

Esto abarca una l??gica realmente interesante, como requerir cierto pago por el contrato para, de esta manera, ejecutar una funci??n.

## Veamos un ejemplo
```
contract OnlineStore {
  function buySomething() external payable {
    // Check to make sure 0.001 ether was sent to the function call:
    require(msg.value == 0.001 ether);
    // If so, some logic to transfer the digital item to the caller of the function:
    transferThing(msg.sender);
  }
}
```

Aqu??, `msg.value` es una manera de ver cuanto Ether fue enviado al contrato, y `ether` es una unidad incorporada.

Lo que sucede aqu?? es que alguien llamar??a a la funci??n desde web3.js (desde la interfaz JavaScript del DApp) de esta manera:

```
// Assuming `OnlineStore` points to your contract on Ethereum:
OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})
```

N??tese el campo `value`, donde la llamada de funci??n javascript especif??ca cu??nto de `ether` enviar (0.001). Si piensas en la transacci??n como un sobre, y los par??metros que usted env??a a la llamada de funci??n son los contenidos de la carta que coloca adentro, entonces a??adir un `value` es como poner dinero en efectivo dentro del sobre ??? la carta y el dinero son entregados juntos al receptor.

>Nota: Si una funci??n no es marcada como `payable` y usted intenta enviar Ether a esta, como se hizo anteriormente, la funci??n rechazar?? su transacci??n.


## Pongalo a prueba

Vamos a crear una funci??n `payable` en nuestro juego zombie.

Digamos que nuestro juego tiene una funci??n donde los usuarios pueden pagar ETH para subir el nivel de sus zombies. El ETH ser?? almacenado en el contrato, el cual usted posee ??? ??Esto es tan solo un ejemplo de c??mo podr??a hacer dinero en sus juego

1. Defina un `uint` llamado `levelUpFee` y configurelo como igual a `0.001 ether`.

2. Cree una funci??n llamada `levelUp`. Esta tomar?? un parametro, `_zombieId`, un `uint`. Deber??a ser `external` y `payable`.

3. La funci??n primero deber??a `require` (requerir) que `msg.value` sea igual a `levelUpFee`.

4. Luego el `level` de este zombie deber??a incrementar: `zombies[_zombieId].level++`.
