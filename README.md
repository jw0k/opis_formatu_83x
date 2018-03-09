# opis_formatu_83x

Każdy z plików z rodziny 83x przechowuje jedną "zmienną". Przez zmienną rozumiemy liczbę, napis, obrazek, program, ustawienia okna, itp. Wyjątkiem jest 83g, który przechowuje grupę zmiennych, czyli więcej niż jedną zmienną.

Każdy z plików 83x ma taki sam format, różni się tylko rozszerzeniem.

|Rozszerzenie  |Opis                      |
|--------------|--------------------------|
|.83b          |TI-83 system backup
|.83b          |TI-83 complex number  <--- (przypuszczalnie tu jest błąd, pewnie powinno być .83c)
|.83d          |TI-83 GDB (function, polar, parametric or sequence)
|.83g          |Multiple TI-83 variables of varying types (group)
|.83i          |TI-83 picture (image)
|.83l          |TI-83 list
|.83m          |TI-83 matrix
|.83n          |TI-83 real number
|.83p          |TI-83 program
|.83s          |TI-83 string
|.83t          |TI-83 table setup
|.83w          |TI-83 window settings (Window or RclWindow)
|.83y          |TI-83 Y-Variable (equation)
|.83z          |TI-83 zoom (saved window settings)


Format pliku 83x przedstawia poniższa tabela. Wszystkie 2-bajtowe liczby są zapisane w little-endian (młodszy bajt pierwszy).

|Offset   |Długość    |Opis
|---------|-----------|-------------|
|0        |8 bajtów   |8-znakowa sygnatura. Zawsze jest to \*\*TI83\*\*
|8        |3 bajty    |0x1A, 0x0A, 0x00
|11       |42 bajty   |Komentarz. Kometarz jest zakończony bajtem zero, albo dopełniony z prawej znakami spacji
|53       |2 bajty    |Długość (w bajtach) sekcji danych w pliku. Długość całego pliku minus 57
|55       |n bajtów   |Sekcja danych
|55+n     |2 bajty    |Checksuma. Dolne 16 bitów sumy wszystkich bajtów w sekcji danych

Sekcja danych składa się z 1 lub więcej wpisów. Każdy wpis opisuje jedną zmienną. Struktura wpisu jest następująca:

|Offset  |Długość   |Opis
|--------|----------|------------------------|
|0       |2 bajty   |0x0B, 0x00
|2       |2 bajty   |Długość (w bajtach) danych zmiennej
|4       |1 bajt    |Type ID zmiennej (zobacz tabela poniżej)
|5       |8 bajtów  |Nazwa zmiennej dopełniona zerami z prawej
|13      |2 bajty   |Długość (w bajtach) danych zmiennej (kopia wartości z offsetu 2)
|15      |m bajtów  |Dane zmiennej


Możliwe wartości Type ID:

|Type ID   |Opis
|----------|----------------|
|00h	     |Real Number
|01h	     |Real List
|02h	     |Matrix
|03h	     |Y-Variable
|04h	     |String
|05h	     |Program
|06h	     |Edit-locked Program
|07h	     |Picture
|08h	     |GDB
|0Ch	     |Complex Number
|0Dh	     |Complex List
|0Fh	     |Window Settings*
|10h	     |Saved Window Settings*
|11h	     |Table Setup*
|13h	     |Backup
|19h	     |Directory* - only used when requesting a directory

*If the Type ID is in the range 0Fh - 12h or 19h, then the contents of the name field of the header do not matter.


UWAGA: w przypadku programów assemblerowych Z80 (Type ID == 5 lub 6) istotne są dwie rzeczy:

1. Dane programu muszą być "unsquishowane", czyli każdy bajt programu powinien być zapisany na dwóch bajtach przy pomocy znaków '0'-'9' i 'A'-'F'. Przykładowo - bajt 0xB7 musi zostać zapisany jako dwa znaki: B7 (a więc 0x42, 0x37)

2. Na końcu programu trzeba dodać "trailer", który składa się z następujących trzech stokenizowanych linii:
    ```
    :End
    :0000
    :End
    ```
    Każda linia kończy się znakiem nowej linii, a także przed pierwszą linią należy dodać znak nowej linii (po ostatnim bajcie kodu). Jest to zatem następujacy ciąg bajtów: `{0x3F,0xD4,0x3F,0x30,0x30,0x30,0x30,0x3F,0xD4}` ("\nend\n0000\nend")

   
Więcej informacji: http://merthsoft.com/linkguide/
