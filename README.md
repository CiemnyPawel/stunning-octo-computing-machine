# [SOI] Projekt 5 - zarządzanie pamięcią
Zarządzanie pamięcią w systemie MINIX za pomocą algorytmu worst-fit

Ćwiczenie 5
Zarządzanie pamięcią


1. Cel ćwiczenia

Domyślnie w systemie Minix algorytmem wyboru wolnego bloku z listy
wolnych bloków, wykorzystywanym do realizacji funkcji systemowych FORK i
EXEC, jest algorytm first fit, czyli wybierany jest pierwszy blok
pamięci o wystarczającym rozmiarze z listy bloków wolnych.

Celem ćwiczenia jest zmiana domyślnego algorytmu przydziału pamięci w
systemie Minix. Należy umożliwić wybór algorytmu wyboru bloku z listy
bloków wolnych między standardowym first fit a tzw. algorytmem worst
fit, czyli takim, w którym wybierany jest blok pamięci z listy wolnych
bloków o największym rozmiarze.

Należy zaimplementować w systemie algorytm worst fit, a następnie
zademonstrować i zinterpretować różnice w działaniu poszczególnych
algorytmów.


2. Zadanie do zrealizowania

Należy zdefiniować dwie dodatkowe funkcje systemowe, identyfikowane stałymi
HOLE_MAP oraz WORST_FIT.


2.1. Funkcja systemowa HOLE_MAP powinna umożliwiać zdefiniowanie własnej
funkcji o sygnaturze:

    int hole_map( void *buffer, size_t nbytes )

która ma za zadanie zwrócić w buforze buffer o rozmiarze nbytes informacje o
aktualnej zawartości listy wolnych bloków utrzymywanej przez moduł
zarządzania pamięcią (MM). Struktura otrzymanej w buforze informacji powinna
być następująca: 
  
  rozmiar1, adres1, rozmiar2, adres2, ..., 0

gdzie kolejne pary rozmiar, adres odpowiadają informacjom o kolejnych
elementach listy wolnych bloków. Rozmiar 0 oznacza ostatni element listy.
Elementy rozmiar i adres mają typ danych unsigned int (na poziomie modułu MM
synonim tego typu o nazwie phys_clicks).

Funkcja hole_map ma zwracać przesłaną liczbę par rozmiar,adres. Należy
zabezpieczyć się przed przepełnieniem zadanego jako argument wywołania
bufora i wypełnić go tylko liczbą par mieszczących się w buforze dbając o
zakończenie listy pozycją rozmiar=0.


2.2. Funkcja systemowa WORST_FIT powinna umożliwiać wybór algorytmu wyboru
elementu z listy wolnych bloków i zdefiniowanie własnej funkcji o
sygnaturze:

    int worst_fit( int w )

która dla w=1 wymusza implementowany w ramach ćwiczenia algorytm przydziału
worst fit, natomiast dla w=0 uaktywnia z powrotem standardowy algorytm first
fit. Wartością zwracaną powinno być zawsze 0.

3. W celu realizacji zadania należy przede wszystkim zapoznać się z
zawartością pliku 

/usr/src/mm/alloc.c

i w pliku tym dodatkowo zaimplementować odpowiednio funkcje:

    PUBLIC int do_worst_fit( void );
    PUBLIC int do_hole_map();

argumenty wejściowe znajdują się w zmiennej globalnej mm_in typu message
przekazanej przez programy użytkowe w wywołaniu _syscall(). Do przekazywania
jako argumentów liczb całkowitych można wykorzystać pole m1_i1 struktury
message, a do przekazania wskazania na bufor pole m1_p1 struktury message.

Do przesyłania zawartości buforów między pamięcią systemu operacyjnego a
pamięcią programów użytkowych należy wykorzystać funkcję sys_copy(), której
przykład użycia można znaleźć w realizacji funkcji systemowej READ
(w pliku /usr/src/fs/read.c).


4. Testowanie funkcjonalności systemu

Należy stworzyć programy użytkowe t.c, w.c oraz x.c z następującą
zawartością:

-------------------------------------------------------------

    /* t.c - polecenie t wyswietla liczbe i rozmiary blokow wolnych */
    #include <stdio.h>
    #include <unistd.h>
    #include <lib.h>
                                                                                    
    PUBLIC int hole_map( void *buffer, size_t nbytes)
    {
    	/* ... _syscall(..HOLE_MAP..) ... */
    }
                                                                                    
    int
    main( void )
    {
            unsigned int    b[1024];
            unsigned int    *p, a, l;
            int     res;
    	    res = hole_map( b, sizeof( b ) );
            printf( "[%d]\t", res );
            p = b;
            while( *p )
            {
                    l = *p++;
                    a = *p++; /* tu niewykorzystywane */
                    printf( "%d\t", l );
            }
            printf( "\n" );
            return 0;
    }
-------------------------------------------------------------
    /* w.c - polecenie w przyjmuje jako argument 1 albo 0 */
    /* wlaczajac/wylaczajac algorytm worst fit w systemie Minix */
    #include <stdlib.h>
    #include <unistd.h>
    #include <lib.h>
    
    
    PUBLIC int worst_fit( int w )
    {
    	/* ... _syscall(..WORST_FIT..) ... */
    }
    
    int
    main( int argc, char *argv[] )
    {
    	if( argc < 2 )
    		return 1;
    	worst_fit( atoi( argv[1] ) );
    	return 0;
    }
-------------------------------------------------------------
    /* x.c - program pomocniczy x, okrojona wersja polecenia sleep */
    /* wykorzystywana do testów */
    #include <stdlib.h>
    #include <unistd.h>
    
    int
    main( int argc, char *argv[] )
    {
    	if( argc < 2 )
    		return 1;
    	sleep( atoi( argv[1] ) );
    	return 0;
    }
-------------------------------------------------------------

Po przygotowaniu powyższych poleceń należy zinterpretować rezultat
działania poniższego skryptu. Do czego służy polecenie chmem (man
chmem)?


    #!/bin/sh
    # skrypt do testowania działania funkcji systemowych 
    # HOLE_MAP oraz WORST_FIT
    cc -o t t.c 
    cc -o w w.c
    cc -o x x.c
    chmem =8000 x
    
    echo "-[ std ]----------------------------------------"
    ./w 0
    for i in 1 2 3 4 5 6 7 8 9 10
    do
    	./x 10 &
    	./t
    	sleep 1
    done
    for i in 1 2 3 4 5 6 7 8 9 10
    do
    	./t
    	sleep 1
    done
    echo "-[ worst ]--------------------------------------"
    ./w 1
    for i in 1 2 3 4 5 6 7 8 9 10
    do
    	./x 10 &
    	./t
    	sleep 1
    done
    for i in 1 2 3 4 5 6 7 8 9 10
    do
    	./t
    	sleep 1
    done
    echo "-[ std ]----------------------------------------"
    ./w 0


5. Uwagi do realizacji zadania

* lista modyfikowanych plików systemowych:
/usr/src/mm/proto.h
/usr/src/mm/alloc.c
/usr/src/mm/table.c
/usr/include/minix/callnr.h

* lista tworzonych programów użytkowych:
w.c
t.c
x.c
skrypt_testowy

* algorytm worst fit wyboru bloku wolnego powinien być zrealizowany jako
fragment funkcji alloc_mem,

* przy realizacji własnego algorytmu wyboru bloku wolnego dopuszczalne
jest zaniedbanie aspektów związanych z wymiataniem (swap),

* należy zwrócić uwagę, że rozmiar bloków przechowywany i wyświetlany
jest nie w bajtach a w jednostkach click,

* należy pamiętać o regularnym zapisywaniu zmian w plikach źródłowych na
osobnej dyskietce,

* w celu poprawy warsztatu pracy w łatwy sposób można powiększyć liczbę
konsoli systemowych z dwóch do np. czterech poprzez zmianę definicji stałej
NR_CONS w pliku /usr/include/minix/config.h

