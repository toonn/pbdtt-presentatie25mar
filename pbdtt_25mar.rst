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

    Word : \bn -> Set
    Word n = Vec Bit n

    split : \all {A} -> (n : \bn) -> (m : \bn) -> Vec A (m \x n) -> Vec (Vec A n) m
    split n zero [] = []
    split n (suc k) xs = (take n xs) \:: (split n k (drop n xs))

    data SplitView {A : Set} : {n : \bn} -> (m : \bn) -> Vec A (m \x n) -> Set where
      [_] : \all {m n} -> (xss : Vec (Vec A n) m) -> SplitView m (concat xss)
    
    view : {A : Set} -> (n : \bn) -> (m : \bn) -> (xs : Vec A (m \x n)) -> SplitView m xs
    view n m xs with concat (split n m xs) | [split n m xs] | splitConcatLemma m xs
    view n m xs | .xs | v | Refl = v

Referenties
===========

::
    
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
