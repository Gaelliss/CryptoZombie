---
title: "Bonus: Kitty Genes"
actions: ['checkAnswer', 'hints']
material:
  editor:
    language: sol
    startingCode:
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

          address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
          KittyInterface kittyContract = KittyInterface(ckAddress);

          // Modify function definition here:
          function feedAndMultiply(uint _zombieId, uint _targetDna) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            // Add an if statement here
            _createZombie("NoName", newDna);
          }

          function feedOnKitty(uint _zombieId, uint _kittyId) public {
            uint kittyDna;
            (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
            // And modify function call here:
            feedAndMultiply(_zombieId, kittyDna);
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
                randDna = randDna - randDna % 100;
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

        address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
        KittyInterface kittyContract = KittyInterface(ckAddress);

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
---

Fonksiyon mant??????m??z ??imdi tamam... fakat bir bonus ??zellik daha ekleyelim.

Onu, kittie'lerden yap??lan zombilerin cat-zombieler oldu??unu g??steren bir benzersiz ??zelli??e sahip olacak ??ekilde yapal??m.

Bunu yapmak i??in, zombinin DNA's??na ??zel bir kitty kodu ekleyebiliriz.

Ders 1'den hat??rlarsan??z, ??u anda zombi g??r??n??m??n?? belirlemek i??in 16 haneli DNA'm??z??n ilk 12 basama????n?? kullan??yoruz. O halde "??zel" karakterleri kullanmak i??in kullan??lmayan son 2 basama???? kullanal??m.

Cat-zombielerin DNA's??n??n son iki basama????n??n `99` oldu??unu farz edece??iz. (kedilerin 9 can?? oldu??undan). Yani kodumuzda, bir zombi bir kediden geliyorsa `if`, o zaman DNA'n??n son iki basama????n?? `99` diyece??iz.

## If ifadeleri

Solidity'de if ifadeleri javascriptteki gibidir:

```
function eatBLT(string sandwich) public {
  // Remember with strings, we have to compare their keccak256 hashes
  // to check equality
  if (keccak256(sandwich) == keccak256("BLT")) {
    eat();
  }
}
```

# Teste koy

Hadi zombi kodumuzdaki kedi genlerini uygulayal??m.

1. ??ncelikle, `feedAndMultiply` i??in fonksiyon tan??m??n?? de??i??tirelim b??ylece o 3. bir arg??man al??r: `_species` isimli bir `string` 

2. Daha sonra, yeni zombinin DNA's??n?? hesaplad??ktan sonra, `_species`'in `keccak256` hashlerini ve `"kitty"` dizisini kar????la??t??ran bir `if` ifadesi ekleyelim

3. `if` ifadesinin i??inde, DNA'n??n son iki basama????n?? `99` ile de??i??tirmek istiyoruz. Bunu yapman??n bir yolu mant??k kullanmak: `newDna = newDna - newDna % 100 + 99;`.

  > A????klama: Varsayal??m `newDna` `334455`'tir. O zaman `newDna % 100` `55`'tir, yani `newDna - newDna % 100` `334400`'d??r. Son olarak  `334499` almak i??in `99` ekleyin.

4. Son olarak, `feedOnKitty` i??inde fonksiyon ??a??r??m??n?? de??i??tirmemiz gerekiyor. `feedAndMultiply`'i ??a????rd??????nda, sonland??rmak i??in `"kitty"` parametresi ekler.
