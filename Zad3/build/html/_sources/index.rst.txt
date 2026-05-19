=========================================
Dokumentacja: 3 Zadanie Laboratoryjne
=========================================

.. toctree::
   :maxdepth: 2
   :caption: Spis treści:

Projekt Bazy Danych: Wypożyczalnia Aut
======================================

Wprowadzenie
============
Wybranym zagadnieniem jest system obsługi wypożyczalni samochodów. Aplikacja służy do gromadzenia informacji o klientach, dostępnej flocie pojazdów (z podziałem na kategorie cenowe) oraz do ewidencjonowania procesu wypożyczeń (rezerwacji).

Opis procesów i towarzyszących im danych
========================================

**Gromadzone dane:**

* **Kategorie:** unikalne ID, nazwa kategorii, cena za dzień.
* **Klienci:** unikalne ID, imię, nazwisko, PESEL, numer prawa jazdy, telefon, e-mail.
* **Samochody:** unikalne ID, marka, model, numer rejestracyjny, rok produkcji, przypisanie do kategorii.
* **Wypożyczenia:** unikalne ID, identyfikator klienta, identyfikator samochodu, data rozpoczęcia, data zakończenia, status rezerwacji.

**Opis procesów i więzy integralności:**

1. **Rejestracja klienta:** Zapis danych osobowych. Pola takie jak PESEL i numer prawa jazdy są objęte więzami unikalności (UNIQUE).
2. **Dodanie pojazdu do floty:** Pojazd przypisywany jest do zdefiniowanej wcześniej kategorii cenowej (klucz obcy). Numer rejestracyjny musi być unikalny.
3. **Proces wypożyczenia:** System tworzy relację między klientem a pojazdem w określonym przedziale czasu. Występuje tu więz integralności (CHECK) wymuszający, aby data zakończenia wypożyczenia była większa lub równa dacie rozpoczęcia.

Prototypowy plik JSON i CSV
===========================

**Format JSON:**

.. code-block:: json

   {
       "id_klienta": 1,
       "imie": "Jan",
       "nazwisko": "Kowalski",
       "pesel": "90051412345",
       "nr_prawa_jazdy": "12345/67/8901",
       "telefon": "+48123456789",
       "email": "jan.kowalski@email.com"
   }

**Format CSV:**

.. code-block:: text

   id_klienta,imie,nazwisko,pesel,nr_prawa_jazdy,telefon,email
   1,Jan,Kowalski,90051412345,12345/67/8901,+48123456789,jan.kowalski@email.com

Dobór typów danych dla wybranych silników
-----------------------------------------

.. list-table:: 
   :widths: 30 35 35
   :header-rows: 1

   * - Atrybut / Pole
     - Typ w PostgreSQL
     - Typ w SQLite
   * - **Klucze Główne**
     - ``SERIAL`` / ``INT GENERATED ALWAYS``
     - ``INTEGER PRIMARY KEY``
   * - **Teksty (Imię, Nazwisko)**
     - ``VARCHAR(X)``
     - ``TEXT``
   * - **Kody i ID (PESEL)**
     - ``VARCHAR(X)``
     - ``TEXT``
   * - **Rok produkcji**
     - ``SMALLINT``
     - ``INTEGER``
   * - **Cena**
     - ``NUMERIC(10, 2)``
     - ``REAL``
   * - **Daty**
     - ``DATE``
     - ``TEXT``

Model koncepcyjny, logiczny i normalizacja
==========================================

W systemie zidentyfikowano encje: **Klient** oraz **Samochód**, które łączą się relacją Wiele-do-Wielu (M:N). Aby uniknąć błędów projektowych (np. pułapek połączeń), zdefiniowano encję asocjacyjną **Wypożyczenie**, rozbijającą tę relację na dwie relacje Jeden-do-Wielu (1:N). 

W celu zachowania poprawności danych bazę doprowadzono do **Trzeciej Postaci Normalnej (3NF)**. Wydzielono tabelę **Kategorie**, dzięki czemu atrybut ceny zależy bezpośrednio od kategorii, a nie od konkretnego egzemplarza samochodu.

Model fizyczny bazy danych
==========================

1. Implementacja w PostgreSQL
-----------------------------

.. code-block:: sql

   CREATE TABLE Kategorie (
       ID_Kategorii INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
       Nazwa VARCHAR(50) NOT NULL,
       Cena_Za_Dzien NUMERIC(8, 2) NOT NULL
   );

   CREATE TABLE Klienci (
       ID_Klienta INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
       Imie VARCHAR(50) NOT NULL,
       Nazwisko VARCHAR(100) NOT NULL,
       PESEL VARCHAR(11) UNIQUE NOT NULL,
       Nr_Prawa_Jazdy VARCHAR(20) UNIQUE,
       Telefon VARCHAR(15),
       Email VARCHAR(100)
   );

   CREATE TABLE Samochody (
       ID_Samochodu INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
       Marka VARCHAR(50) NOT NULL,
       Model VARCHAR(50) NOT NULL,
       Nr_Rejestracyjny VARCHAR(15) UNIQUE NOT NULL,
       Rok_Produkcji SMALLINT CHECK (Rok_Produkcji > 1900),
       ID_Kategorii INT,
       CONSTRAINT fk_kategoria FOREIGN KEY (ID_Kategorii) REFERENCES Kategorie(ID_Kategorii)
   );

   CREATE TABLE Wypozyczenia (
       ID_Wypozyczenia INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
       ID_Klienta INT NOT NULL,
       ID_Samochodu INT NOT NULL,
       Data_Od DATE NOT NULL,
       Data_Do DATE NOT NULL,
       Status VARCHAR(20) DEFAULT 'Zarezerwowane',
       CONSTRAINT fk_klient FOREIGN KEY (ID_Klienta) REFERENCES Klienci(ID_Klienta),
       CONSTRAINT fk_samochod FOREIGN KEY (ID_Samochodu) REFERENCES Samochody(ID_Samochodu),
       CONSTRAINT sprawdz_daty CHECK (Data_Do >= Data_Od)
   );

2. Implementacja w SQLite
-------------------------

.. code-block:: sql

   CREATE TABLE Kategorie (
       ID_Kategorii INTEGER PRIMARY KEY,
       Nazwa TEXT NOT NULL,
       Cena_Za_Dzien REAL NOT NULL
   );

   CREATE TABLE Klienci (
       ID_Klienta INTEGER PRIMARY KEY,
       Imie TEXT NOT NULL,
       Nazwisko TEXT NOT NULL,
       PESEL TEXT UNIQUE NOT NULL,
       Nr_Prawa_Jazdy TEXT UNIQUE,
       Telefon TEXT,
       Email TEXT
   );

   CREATE TABLE Samochody (
       ID_Samochodu INTEGER PRIMARY KEY,
       Marka TEXT NOT NULL,
       Model TEXT NOT NULL,
       Nr_Rejestracyjny TEXT UNIQUE NOT NULL,
       Rok_Produkcji INTEGER,
       ID_Kategorii INTEGER,
       FOREIGN KEY (ID_Kategorii) REFERENCES Kategorie(ID_Kategorii)
   );

   CREATE TABLE Wypozyczenia (
       ID_Wypozyczenia INTEGER PRIMARY KEY,
       ID_Klienta INTEGER,
       ID_Samochodu INTEGER,
       Data_Od TEXT NOT NULL,
       Data_Do TEXT NOT NULL,
       Status TEXT NOT NULL,
       FOREIGN KEY (ID_Klienta) REFERENCES Klienci(ID_Klienta),
       FOREIGN KEY (ID_Samochodu) REFERENCES Samochody(ID_Samochodu)
   );
