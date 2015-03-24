Programmeren in Dependently Typed Talen
=======================================
Toon Nolten

.. image:: image/cc_by.png


Overzicht
=========
* Totality & Turing completeness
* DSEL

  * Cryptol
  * Relational Algebra


Totality & Turing completeness
==============================

* Talen met dependent types hebben een totality checker zodat type checking
  gegarandeerd eindigt
* "Totale functies eindigen dus algemene recursie is niet mogelijk dus zo'n
  taal is niet Turing compleet."
* Totaal is niet hetzelfde als terminerend, voor functies op codata betekent
  het productief
* Codata laat ons toe Turing machines te simuleren, talen met dependent types
  kunnen dus wel Turing compleet zijn


DSEL
====

* Beter geschikt voor taken in het domein
* Gebruikers moeten een nieuwe taal leren
* Geen nieuwe taal, bij een DSL wel
* Alle abstracties uit de general purpose language zijn beschikbaar
* Om een DSL te kunnen embedden moet ook het type systeem geëmbed worden,
  de meeste type systemen zijn hier niet krachtig genoeg voor


Cryptol
=======

* DSL voor cryptografische protocols
* Oorspronkelijk voor de NSA
* SIMON & SPECK

.. code-block:: cryptol

    x : [8]; x = 42;

.. code-block:: cryptol

    swab : [32] -> [32];
    swab [a b  c d] = [b a c d];

.. code-block:: agda

    Word : ℕ → Set
    Word n = Vec Bit n

    split : ∀ {A} → (n : ℕ) → (m : ℕ) → Vec A (m × n) → Vec (Vec A n) m
    split n zero [] = []
    split n (suc k) xs = (take n xs) ∷ (split n k (drop n xs))

    data SplitView {A : Set} : {n : ℕ} → (m : ℕ) → Vec A (m × n) → Set where
      [_] : ∀ {m n} → (xss : Vec (Vec A n) m) → SplitView m (concat xss)
    
    view : {A : Set} → (n : ℕ) → (m : ℕ) → (xs : Vec A (m × n)) → SplitView m xs
    view n m xs with concat (split n m xs) | [split n m xs] | splitConcatLemma m xs
    view n m xs | .xs | v | Refl = v


Relational Algebra
==================

* Databases zijn enorm belangrijk, zo goed als alles op het internet komt uit
  een database
* Het is belangrijk voor een programmeertaal om een goede interface tot
  databases te hebben
* De meeste RDMS systemen verwachten een SQL query in een string en geven ofwel
  een string terug, ofwel is het type van het antwoord enkel dynamisch bepaald


Relational Algebra
==================
Bestaande interfaces
--------------------

* Een request functie die een string verwacht

  * Dit is unsafe, er is geen enkele vorm van statische checks
  * Een syntactisch foute of semantisch onzinnige query leidt tot een
    runtime error
  * SQL is een extra taal die geleerd dient te worden

* Voor Haskell zijn er verschillende voorstellen gedaan om dit te verbeteren,
  maar

  * Sommige concepten uit relationele algebra zoals *join* en
    *cartesisch product* zijn bijzonder moeilijk te typeren
  * Het type systeem van Haskell is niet krachtig genoeg dus zijn er vaak
    extensies nodig die eigenlijk niet tot Haskell behoren
  * Als de binding *safe* is, is er gewoonlijk een preprocessor nodig die
    de types genereert voor een specifieke database

* Veel van de populaire bindings geven type safety op in ruil voor een handiger
  gebruik


Relational Algebra
==================
Dependently typed
-----------------

* Volledig *safe*, dus statische garantie dat een query goed gevormd is en
  een antwoord van het juiste type zal teruggeven (als de connectie niet
  wegvalt...)
* *Totally embedded*, er is dus geen enkele vorm van preprocessor nodig
* De code is een stuk eenvoudiger dan die van de type-safe Haskell bindings


Relational Algebra
==================

* Voor we queries kunnen uitvoeren moeten we eerst verbinden met een database,
  in Haskell gebeurt dit gewoonlijk als volgt:

  .. code-block:: haskell

      connect :: ServerName -> IO Connection

* Een probleem hiermee is dat het geen enkele statische garantie kan bieden
  over het type van resultaten van queries die je uitvoert m.b.v. die Connection
* Met dependent types kunnen we veel preciezer zijn:

  .. code-block:: Agda

      Handle : Schema → Set
      connect : ServerName → TableName → (s : Schema) → IO (Handle s)

* Dit zorgt dat we statische garanties hebben over welke queries we kunnen
  uitvoeren en wat het antwoord daarop kan zijn
  
  * Er kan nog steeds een fout gebeuren bij het aanmaken van de verbinding,
    als het schema niet overeenkomt met dat van de tabel
  * Als het schema wel klopt, kan er niets misgaan met het programma
    (het wegvallen van de verbinding, verandering van het schema in de database,
    enz. daargelaten)


Relational Algebra
==================

.. code-block:: Agda

    data RA : Schema → Set where
      Read : ∀ {s} → Handle s → RA s
      Union : ∀ {s} → RA s → RA s → RA s
      Diff : ∀ {s} → RA s → RA s → RA s
      Product : ∀ {s s'} → {_ : So (disjoint s s')} → RA s → RA s' → RA (append s s')
      Project : ∀ {s} → (s' : Schema) → {_ : So (sub s' s)} → RA s → RA s'
      Select : ∀ {s} → Expr s BOOL → RA s → RA s


Referenties
===========

::
    
    Totality versus Turing completeness
        https://github.com/pigworker/Totality
          /blob/master/Totality-slides.pdf

    The Power of Pi
        http://cs.ru.nl/~wouters/Publications
          /ThePowerOfPi.pdf

    Cryptol (Galois, Inc.)
        https://galois.com/project/cryptol/

    SIMON and SPECK: new NSA Encryption Algorithms
        https://www.schneier.com/blog/archives
          /2013/07/simon_and_speck.html

    SIMON and SPECK in Cryptol
        http://galois.com/blog/2013/06
          /simon-and-speck-in-cryptol/
