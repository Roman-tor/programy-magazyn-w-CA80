Po napisaniu jakiegoś programu do CA80, jeśli chcemy go gdzieś zapamietać/zapisać /na nośniku danych oczywiście/, możemy użyć funkcji standardowej w CA80 czyli zapis na magnetofon , zlec. #4 w MONITORZE CA80. Alternatywą do tego zapisu na taśmie magnetofonewej jest zapis np. do pamięci IIC typu AT24C512. Jest to pamięc EEPROM, którą można zapisywać i odczytywać bezpośrednio za pomocą CA80, używając programu np. z mojego repozytorium "AT24C512_I2C_eeprom". Mieści się tam sporo programów w przestrzeni  64KB. Problem może być tylko, jeśli mamy więcej programów i chcielibyśmy mieć do nich łatwy dostęp. Dlatego opracowałem program  zapisu i odczytu programów z CA80 na AT24C512 i na odwrót.

Można również wykorzystać to:
https://microgeek.eu/viewtopic.php?f=82&t=2435#p14642
ale tu potrzebna jest płytka PCB z pinami i wkładana do CA80 tam gdzie pamięc od 4000h-7FFFh, ta pamięc SST39SF040 ma aż 4Mbit !!! Jest tam też program do obsługi tej płytki. Ja ma do mojej plytki - tej na zdjęciu, trochę inny program, w którym też jest LCD 20x4 i możemy zapis do FLASH robić w dowolnym miejscu, nie tylko od początku sektora / sektor liczy 4 kb - 0-FFF/.

Moja wersja tej płytki wygląda nieco inaczej- nie trzeba wkładać w podstawkę tylko do bocznego złącza 34-pin lub 40-pin, w zależności od płytki MIK 290 - można dolutować płaski kabel do pinów podstawki U10. Jeśli wlozymy tę płytkę, MUSIMY wyjąc pamięć z podstawki U10!!!
Inna wersja MIK 290 #phill2k na 
https://microgeek.eu/viewtopic.php?f=82&t=2227
ma już takie gniazdo /40-pin/, tylko trzeba doprowadzić sygnał CE_ do pinu 20 na płytce /patrz MOD-4/, ja w mojej wersji tak zrobiłem. W mojej wersji, po zmianie jumperów J2, J3 i J4 i programu, można zapisywać pamięć 28C64-256

![płytka FLASH](https://github.com/user-attachments/assets/9889acc2-78af-43b0-8375-c0d04c20b75a)

Mój program jest dość długi /ok. 2.800 bajtow/ ale łatwy w obsłudze; warunkiem dobrej przejrzystości programu jest wykorzystanie wyświetlacza LCD 20x4, który też jest w moim repozytirum /schemat podłączenia opracował #ZEGAR, podłączony pod port systemowy U7 -8255. Wyświetlacz pozwoli nam wyświetlać nr i nazwę szukanych programów /nazwę programu musimy wpisać podczas pisania programu w kompilatorze / poprzedzając ją znakami DDE2h/.
Moje programy, a mam ich teraz ok. 20, mają /przeważnie na końcu programu/ tę nazwę, np "KALENDARZ". Pamięć AT24C512 jest podłączona tylko za pomocą 4. przewodów: zasilanie do odpowiednich pinów złącza ZU50: Vcc do pinu 48, GND do pinu 50, linia DATA do portu PB.0 - pin 9 a linia CLK do portu PC.4 - pin 22 układu U21 - 8255 złącza użytkownika. Pamięć jest ustawiona na "adres" A0 czyli piny AT28C512- A0, A1 i A2 /nóżki 1, 2 i 3/ tejże pamięci są połączone do GND. Oczywiście, można to zmienić, nadać inny adres ale wówczas w pliku ASM musimy wpisać odpowiedni adres pod <adr_z_64>  np. na A4 / <adr_o_64> jest obliczany automatycznie/.
Możemy podlączyć linię DATA do innego portu, np PA.0 lub PC.0 ale musimy wtedy wybrać w pliku ASM odpowiednie ustawienia - linie 62 do 80 w tym pliku. Ale linia CLK powinna być podłaczona do PC.4. Jeśli chciałabyś/ chciałbyś podłączyć DATA lub CLK do innych linii portu PA, PB lub PC, napisz do mnie e-mail, spróbuję to zmienić.
Schemat podłaczenia iiC do CA80

![iiC_schemat](https://github.com/user-attachments/assets/95cde02a-e141-4838-89ac-db00536a8bf7)

Nową pamięć EEPROM iiC, musimy "zainicjować" zleceniem *0 : powoduje ono wpisanie od adr. 1000h FE FE... a od 1100h FD E4 FE FE. Od tego momemtu mamy pamięć przygotowaną do wpisywania naszych programów /opis też w pliku ASM/

Pamięc AT24C512 pod nasze programy jest podzielona jakby na trzy części:
0-FFFh to miejsce na program obsługi, abyśmy mogli go wczytać do RAM CA80 i uruchomić.
1000h-10FFh -  na numery programów i ich umiejscowienie w EEPROM iiC
1100h - FFFFh - nasze programy, każdy program zaczyna się od znacznika FDE4, potem numer programu, /1. bajt/ i następne dwa bajty to początek programu w CA80, i dalej już sam program / pamiętaj aby w programie (podczas kompilacji) nadać mu nazwę po znaczniku DDE2h ! Nazwa kończy się FFh -1x/. Jeśli nazwy nie będzie, zostanie wyświetlony komunikat "bez nazwy" po numerze programu. Program wpisany do EEPROM kończy się bajtami FFFF..., ok. 24 /"dokładanie" automatycznie po wpisie programu do pamięci iiC/- to miejsce na ewentualne poprawki programu, abyśmy nie musieli zmieniać całego układu pamięci. Po tych FFFF... znowu FDE4, nr programu, pocz. w CA80 itd, j.w.
Sektor drugi /1000-10FF/ to numery programów i adresy w EEPROM, też wpisywane automatycznie podczas wpisywania jakiegoś programu z CA80 do EEPROM iiC. Pierwszy bajt to nr programu, po nim dwa bajty oznaczające początek tego programu w EEPROM, natęonie znomu nr programu, adres w EEPROM itd, po ostatnim programie FE FE FE aż do 10FFh. Jeśli wpiszemy jakiś program, to automatycznie zostanie "dopisany" do obszaru 1000-10FF, za ostatnim programem. Jest to opisane w pliku ASM i LST.
Po uruchomieniu programu, sektor ten jest przepisywany do RAM CA80, od FE10h.
A tak wygląda ekran startowy na LCD i na CA80, po uruchomieniu programu:

![ekran_powitalny](https://github.com/user-attachments/assets/b901c505-e743-4d7c-ba89-163293d49598)

ZLECENIE # 0 - to kasowanie: pamięci lub programu; kasowanie pamięci jest opisane wyżej, kasowanie jakiegoś programu polega na znalezieniu danego programu w EEPROM iiC i wpisaniu po FD E4  bajtów FF - oznacza to "pusty "obszar czyl wypełniony bajtami FF aż do następnego początku programu; a w obszarze od 1000h w EEPROM,  w miejsce numeru programu / i adresu w EEPROM/ wpisanie trzech bajtów FF. To samo/ czyli kasowanie programu/ możemy zrobić zlec. # 4 - "zapisz obszar", wpisując uprzednio w obszarze RAM CA80 odpowiednie bajty FF FF.. i przepisać ten obszar do EEPROM. Miejsce "położenia" programu odczytamy zlec. # 5 - DŁUGOŚĆ, musimy to oczywiście zrobic przed skasowaniem danego programu

ZLECENIE # 1 - szukanie wolnego numeru programu; po wcisnięciu klawisza 1, program znajdzie nam pierwszy wolny numer programu, który możemy nadać naszemu nowemu programowi

![wolny_progr](https://github.com/user-attachments/assets/b9bd8ca7-5a8b-494d-a1a2-50ec2ade84ed)


ZLECENIE # 2 AKTUALIZACJA - powoduje przepisanie obszaru 1000h-10FFh z EEPROM do RAM cA80 od FE10h

ZLECENIE # 4  ZAPISZ - możemy zapisać program - klawiszem A, z obszaru RAM [od ] [.] [ do ] [.] [NR] [=]  (nr to nadany przez nas numer programu); jesli wcisniemy klawisz E, to możemy przepisać obszar z CA80 do EEPORM [od CA] [.] [do_CA] [.] od_EEP] [=]

![zapisz_progr](https://github.com/user-attachments/assets/91097745-be70-4a2d-b28b-cd332984e24a)


ZLECENIE # 5 DŁUGOŚĆ - musimy podać nr szukanego programu - na CA80 wyświetli się długość programu / tylko tyle bo mamy do dyspozycji 8. znaków a na LCD wyświetli się nam 

![dlug_progr](https://github.com/user-attachments/assets/8444bff1-aa5a-44a8-9696-5a6e6dc8f0ea)

ZLECENIE # 6 SZUKANIE PROGRAMU - po wcisnieciu klawisza 6, są wyszukiwane programy znajdujące sie w EEPROM i ich numery jak i nazwa są wyśwuetlane na LCD, na CA80 adres pocz. denego programu; wyśietlane są po cztery programy /ze względu na lCD/, jeśli wcisniemy klawisz = , szukane są nastęone programy; jeśli wciśniemy klawisz 0-9 - tzn. jakiś nr programu, zostanie on odszukany, przepisany do CA80 i uruchomiony.

![szukanie_progr](https://github.com/user-attachments/assets/a6a46d7b-27d5-43b8-bdfe-552716b820b5)








cdn...
