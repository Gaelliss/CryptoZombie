---
title: Gas
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiefactory.sol": |
        pragma solidity ^0.4.25;

        import "./ownable.sol";

        contract ZombieFactory is Ownable {

            event NewZombie(uint zombieId, string name, uint dna);

            uint dnaDigits = 16;
            uint dnaModulus = 10 ** dnaDigits;

            struct Zombie {
                string name;
                uint dna;
                // Aggiungi nuovi dati qui
            }

            Zombie[] public zombies;

            mapping (uint => address) public zombieToOwner;
            mapping (address => uint) ownerZombieCount;

            function _createZombie(string _name, uint _dna) internal {
                uint id = zombies.push(Zombie(_name, _dna)) - 1;
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

          function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
            require(msg.sender == zombieToOwner[_zombieId]);
            Zombie storage myZombie = zombies[_zombieId];
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
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
        pragma solidity ^0.4.25;

        /**
        * @title Ownable
        * @dev Il contratto di propriet?? ha un indirizzo del proprietario e fornisce funzioni di controllo
        * delle autorizzazioni di base, ci?? semplifica l'implementazione delle "autorizzazioni dell'utente".
        */
        contract Ownable {
          address private _owner;

          event OwnershipTransferred(
            address indexed previousOwner,
            address indexed newOwner
          );

          /**
          * @dev Il costruttore di propriet?? imposta il `proprietario` originale del contratto sull'account del mittente.
          */
          constructor() internal {
            _owner = msg.sender;
            emit OwnershipTransferred(address(0), _owner);
          }

          /**
          * @return l'indirizzo del proprietario.
          */
          function owner() public view returns(address) {
            return _owner;
          }

          /**
          * @dev Genera se chiamato da qualsiasi account diverso dal proprietario.
          */
          modifier onlyOwner() {
            require(isOwner());
            _;
          }

          /**
          * @return vero se `msg.sender` ?? il proprietario del contratto.
          */
          function isOwner() public view returns(bool) {
            return msg.sender == _owner;
          }

          /**
          * @dev Consente all'attuale proprietario di rinunciare al controllo del contratto.
          * @notice La rinuncia alla propriet?? lascer?? il contratto senza un proprietario
          * Non sar?? pi?? possibile chiamare le funzioni con il modificatore `onlyOwner`.
          */
          function renounceOwnership() public onlyOwner {
            emit OwnershipTransferred(_owner, address(0));
            _owner = address(0);
          }

          /**
          * @dev Consente all'attuale proprietario di trasferire il controllo del contratto a un nuovo proprietario.
          * @param newOwner L'indirizzo a cui trasferire la propriet??.
          */
          function transferOwnership(address newOwner) public onlyOwner {
            _transferOwnership(newOwner);
          }

          /**
          * @dev Trasferisce il controllo del contratto a un nuovo proprietario.
          * @param newOwner L'indirizzo a cui trasferire la propriet??.
          */
          function _transferOwnership(address newOwner) internal {
            require(newOwner != address(0));
            emit OwnershipTransferred(_owner, newOwner);
            _owner = newOwner;
          }
        }
    answer: >
      pragma solidity ^0.4.25;

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
---

Grande! Ora sappiamo come aggiornare porzioni chiave della DApp impedendo ad altri utenti di incasinare i nostri contratti.

Diamo un'occhiata ad un altro modo per cui Solidity ?? molto diversa dagli altri linguaggi di programmazione:

## Gas - il carburante Ethereum che funziona con le DApps 

In Solidity i tuoi utenti devono pagare ogni volta che eseguono una funzione sulla DApp usando una valuta chiamata **_gas_**. Gli utenti acquistano gas con Ether (la valuta su Ethereum), quindi i tuoi utenti devono spendere ETH per eseguire le funzioni sulla tua DApp.

La quantit?? di gas necessaria per eseguire una funzione dipende dalla complessit?? della logica di tale funzione. Ogni singola operazione ha un **_costo del gas_** basato approssimativamente sulla quantit?? di risorse di elaborazione necessarie per eseguire tale operazione (ad es. La scrittura in memoria ?? molto pi?? costosa dell'aggiunta di due numeri interi). Il **_costo del gas_** totale della tua funzione ?? la somma dei costi del gas di tutte le sue singole operazioni.

Poich?? l'esecuzione di funzioni costa denaro reale per i tuoi utenti, l'ottimizzazione del codice ?? molto pi?? importante in Ethereum che in altri linguaggi di programmazione. Se il tuo codice ?? pessimo, i tuoi utenti dovranno pagare di pi?? per eseguire le tue funzioni ??? e questo potrebbe aggiungere fino a milioni di dollari in commissioni non necessarie tra le migliaia di utenti.

## Perch?? ?? necessario il Gas?

Ethereum ?? come un grande, lento, computer ma estremamente sicuro. Quando si esegue una funzione ogni singolo nodo sulla rete dovr?? eseguire quella stessa funzione per verificarne l'output: migliaia di nodi che verificano l'esecuzione di ogni funzione sono ci?? che rende decentralizzato Ethereum ed i suoi dati immutabili e resistenti alla censura.

I creatori di Ethereum volevano assicurarsi che qualcuno non potesse ostruire la rete con un ciclo infinito, o impegnare tutte le risorse della rete con calcoli davvero pesanti. Quindi lo hanno fatto in modo che le transazioni non siano gratuite e gli utenti debbano pagare per i tempi di calcolo e per lo spazio di archiviazione.

> Nota: questo non ?? necessariamente vero per i sidechains, come quelli che gli autori di CryptoZombies stanno costruendo su Loom Network. Probabilmente non avr?? mai senso eseguire un gioco come World of Warcraft direttamente sulla rete principale di Ethereum: i costi del gas sarebbero proibitivi. Ma potrebbe funzionare su una sidechain con un diverso algoritmo di consenso. Parleremo di pi?? su quali tipi di DApps vorresti distribuire su sidechains rispetto alla mainnet di Ethereum in una lezione futura.

## Struttura ad incastro per risparmiare gas

Nella Lezione 1 abbiamo menzionato che ci sono altri tipi di `uint`s: `uint8`, `uint16`, `uint32`, ecc.

Normalmente non c'?? alcun vantaggio nell'utilizzare questi sottotipi perch?? Solidity riserva 256 bit di memoria indipendentemente dalla dimensione `uint`. Ad esempio, l'uso di `uint8` invece di `uint` (`uint256`) non ti far?? risparmiare alcun gas.

Ma c'?? un'eccezione a questo: dentro `struct`s.

Se hai pi?? `uint`s all'interno di una struttura, l'uso di un `uint` di dimensioni minori, quando possibile, consentir?? a Solidity di raggruppare queste variabili per occupare meno spazio di archiviazione. Per esempio:

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

// `mini` coster?? meno gas rispetto a` normale` a causa della struttura ad incastro.
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

Per questo motivo all'interno di una struttura ti consigliamo di utilizzare i sottotipi di numeri interi pi?? piccoli con cui puoi cavartela.

Ti consigliamo inoltre di raggruppare insieme tipi di dati identici (ovvero metterli uno
accanto all'altro nella struttura) in modo che Solidity possa ridurre al minimo lo spazio di archiviazione richiesto. Ad esempio, una struttura con
campi `uint c; uint32 a; uint32 b;` coster?? meno gas di una struttura con campi `uint32 a; uint c; uint32 b;`
perch?? i campi `uint32` sono raggruppati insieme.


## Facciamo una prova

In questa lezione aggiungeremo 2 nuove funzionalit?? ai nostri zombi: `level` e `readyTime` ??? quest'ultimo verr?? usato per implementare un timer di ricarica per limitare la frequenza con cui uno zombi pu?? nutrirsi.

Quindi torniamo a `zombiefactory.sol`.

1. Aggiungi altre due propriet?? alla nostra struttura `Zombie`: `level` (un `uint32`) e `readyTime` (anch'esso un `uint32`). Vogliamo mettere insieme questi tipi di dati, quindi mettiamoli alla fine della struttura.

32 bit ?? pi?? che sufficiente per contenere il livello ed il timestamp dello zombi, quindi questo ci far?? risparmiare alcuni costi del gas comprimendo i dati pi?? strettamente rispetto all'uso di un normale `uint` (256 bit).
