---
title: Spare Gas med 'View' funksjoner
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiehelper.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefeeding.sol";

        contract ZombieHelper is ZombieFeeding {

          modifier aboveLevel(uint _level, uint _zombieId) {
            require(zombies[_zombieId].level >= _level);
            _;
          }

          function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
            require(msg.sender == zombieToOwner[_zombieId]);
            zombies[_zombieId].name = _newName;
          }

          function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
            require(msg.sender == zombieToOwner[_zombieId]);
            zombies[_zombieId].dna = _newDna;
          }

          // Lag funksjonen her

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
      "zombiefactory.sol": |
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

      import "./zombiefeeding.sol";

      contract ZombieHelper is ZombieFeeding {

        modifier aboveLevel(uint _level, uint _zombieId) {
          require(zombies[_zombieId].level >= _level);
          _;
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

        }

      }
---

R??tt! N?? har vi noen spesielle evner for h??yere niv?? zombier, for ?? gi eierne et insentiv til ?? levle dem opp. Vi kan legge til flere av disse senere hvis vi vil.

La oss legge til en ekstra funksjon: v??r DApp trenger en metode for ?? se en brukers zombie-h??r - la oss kalle det `getZombiesByOwner`.

Denne funksjonen trenger bare ?? lese data fra blockchain-en, s?? vi kan gj??re det til en `view`-funksjon. Som bringer oss til et viktig tema n??r vi snakker om gassoptimalisering:
## View funksjoner koster ikke gas

`view` Funksjoner koster ingen gas n??r de kalles eksternt av en bruker.

Dette skyldes at "view" -funksjonene ikke endrer noe p?? blockchain - de leser bare dataene. S?? en funksjon market `view` forteller `web3.js` at den bare trenger ?? sp??rre din lokale Ethereum-node for ?? kj??re funksjonen, og det trenger ikke faktisk ?? opprette en transaksjon p?? blockchain (som m??tte kj??re p?? hver eneste knutepunkt, og koster gas).

Vi dekker ?? sette opp web3.js med din egen node senere. Men for n?? er det du b??r ta med deg at du kan optimalisere DApps gas-bruk for brukerne dine, ved ?? bruke skrivebeskyttede `external view`-funksjoner, hvor det er mulig.

> Noter: Hvis en `view`-funksjon kalles internt fra en annen funksjon i samme kontrakt som er **ikke** en `view`-funksjon, vil den fortsatt koste gas. Dette skyldes at den andre funksjonen skaper en transaksjon p?? Ethereum, og vil fortsatt bli verifisert fra hver knutepunkt. S?? `view`-funksjoner er bare gratis n??r de kalles eksternt.

## Test det

Vi skal implementere en funksjon som vil returnere brukerens hele zombie h??r. Vi kan senere kj??re denne funksjonen fra `web3.js` hvis vi vil vise en brukerprofilside med hele h??ren.

Denne funksjonens logikk er litt komplisert, s?? det vil ta noen f?? kapitler ?? implementere.

1. Lag en ny funksjon som heter `getZombiesByOwner`. Det vil ta ett argument, en `address` som heter`_owner`.

2. La oss gj??re det til en `external view`-funksjon, s?? vi kan kalle det fra `web3.js` uten ?? trenge noen gas.

3. Funksjonen skal returnere en `uint []` (en array av `uint`).

La funksjonskroppen v??re tom for n??, vi fyller den inn i neste kapittel.
