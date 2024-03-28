Hay que centrarse siempre en los errores y ignorar las notas. Los errores que salen al principio son mucho más importantes que los que salen al final.

# Función no definida
La parte importante es: Undefined reference to <función>
```
/usr/bin/ld: /usr/lib/gcc/x86_64-linux-gnu/11/../../../x86_64-linux-gnu/Scrt1.o: in function `_start':
(.text+0x1b): undefined reference to `main'
``` 

## Ejemplos
Cuando no hay una función main.
```cpp
#include <iostream>
using namespace std;

int areaCuadrado(int base, int altura) {
  return base * altura;
}
```

Cuando llamamos a una función que no está implementada: 
```cpp 
#include <iostream>
using namespace std;

int areaCuadrado(int base, int altura);

int main() {
  areaCuadrado(30,20);
  return 0;
}
```

El error:
```cpp
/usr/bin/ld: /tmp/ccJdmek6.o: in function `main':
test.cpp:6: undefined reference to `areaCuadrado(int, int)'
collect2: error: ld returned 1 exit status
```

Cuando no implementamos un método:
```cpp 
#include <iostream>
#include <string>
using namespace std;

class Persona {
public:
  Persona();
  std::string nombre;
  int edad;
  
};

int main() {
  Persona juan;
  return 0;
}
```

``` 
/usr/bin/ld: /tmp/cc3yebR3.o: in function `main':
test.cpp:14: undefined reference to `Persona::Persona()'
```

# Función definida varias veces
En un fichero .cpp no se puede definir dos veces la misma función.
```cpp
#include <iostream>

int areaCuadrado(int base, int altura) {
  return base * altura;
}

int areaCuadrado(int base, int altura) {
  return 42;
}

int main() {
  areaCuadrado(30,20);
  return 0;
}
```
El error:
``` 
test.cpp:9:5: error: redefinition of ‘int areaCuadrado(int, int)’
    9 | int areaCuadrado(int base, int altura) {
      |     ^~~~~~~~~~~~
test.cpp:5:5: note: ‘int areaCuadrado(int, int)’ previously defined here
    5 | int areaCuadrado(int base, int altura) {
      |     ^~~~~~~~~~~~
```
Nos dice las líneas en las que pasa, los archivos y nos da una nota para añadir más información. La parte importante es el primer `error`.
Este error también puede ocurrir cuando usamos archivos .h y nos olvidamos de poner `#pragma once` en la primera línea o `#ifndef TEST_H #define TEST_H #endif`.


Otro caso de error de redefinición de funciones se da cuando la misma función está implementada en dos archivos .cpp diferentes. Esto puede pasar cuando defines una función en un [fichero de encabezado header](https://gcc.gnu.org/onlinedocs/cpp/Header-Files.html). Cuando se usa `#include "archivo.h"` se copian los contenidos de ese archivo.h al fichero que queremos. Si una función está implementada en un archivo .h y incluyes ese archivo en dos archivos .cpp diferentes dará error.

El archivo test.cpp:
```cpp
#include "test.h"

int main() {
  areaCuadrado(30,20);
  return 0;
}
```
El archivo test2.cpp:
```cpp
#include "test.h"
```
El archivo test.h.
```cpp
#pragma once
int areaCuadrado(int base, int altura) {
  return base * altura;
}
```
El error:
```
/usr/bin/ld: /tmp/ccqCp49p.o: in function `areaCuadrado(int, int)':
test.h:1: multiple definition of `areaCuadrado(int, int)'; /tmp/ccZjMwSj.o:/home/david/org/uni/1/mp/yy/test.h:1: first defined here
```
La línea importante es la segunda, que dice multiple definition.

Para arreglar el problema tenemos que implementar la función en uno de los archivos .cpp y dejar solo la firma de la función en el .h:
test.cpp:
```cpp
#include "test.h"

int main() {
  areaCuadrado(30,20);
  return 0;
}
```
test2.cpp:
```cpp
#include "test.h"
int areaCuadrado(int base, int altura) {
  return base * altura;
}
```
test.h:
```cpp
#pragma once
int areaCuadrado(int base, int altura);
```

# Constructor por defecto imposible
A veces, cuando no definimos un constructor el compilador crea un constructor por defecto por nosotros:
```cpp
#include <string>
using namespace std;
class Persona {
public:
	string m_nombre;
}

int main() {
	Persona vacio;
}
```
Vacio será `{m_nombre = ""}`. Esto se puede hacer cuando todos los miembros de la clase tienen constructores por defecto, saben como construirse solos sin argumentos. Si algún miembro no tiene constructor por defecto, tendremos este error:
```cpp 
#include <iostream>
#include <string>
using namespace std;

class Mascota {
public:
  Mascota(int edad, string raza)
    :m_edad(edad), m_raza(raza)
  {}
  int m_edad;
  string m_raza;
};

class Persona {
public:
  Mascota m_mascota;
};

int main() {
  Persona juan;
  return 0;
}
```
El error: 
``` 
test.cpp: In function ‘int main()’:
test.cpp:20:11: error: use of deleted function ‘Persona::Persona()’
   20 |   Persona juan;
      |           ^~~~
test.cpp:14:7: note: ‘Persona::Persona()’ is implicitly deleted because the default definition would be ill-formed:
   14 | class Persona {
      |       ^~~~~~~
test.cpp:14:7: error: no matching function for call to ‘Mascota::Mascota()’
test.cpp:7:3: note: candidate: ‘Mascota::Mascota(int, std::string)’
    7 |   Mascota(int edad, string raza)
      |   ^~~~~~~
test.cpp:7:3: note:   candidate expects 2 arguments, 0 provided
test.cpp:5:7: note: candidate: ‘Mascota::Mascota(const Mascota&)’
    5 | class Mascota {
      |       ^~~~~~~
test.cpp:5:7: note:   candidate expects 1 argument, 0 provided
test.cpp:5:7: note: candidate: ‘Mascota::Mascota(Mascota&&)’
test.cpp:5:7: note:   candidate expects 1 argument, 0 provided
```
Hay que ignorar las líneas que dicen nota y centrarse en las que dicen error. Los dos errores que hay nos dicen que no se puede llamar al constructor por defecto de Persona (está borrado) y que no existe el constructor por defecto de Mascota.

# Errores tontos
## Miembro de la clase es privado
```
test.cpp:11:8: error: ‘std::string Persona::nombre’ is private within this context
   11 |   persona.nombre = "Juan";
      |        ^~~~~~
```
## Variable no definida
Cuando se te olvida definir una variable o escribes el nombre mal.
``` 
test.cpp:12:11: error: ‘juan’ was not declared in this scope
   12 |   cout << juan << endl;
      |           ^~~~
```

## Type mismatch
Cuando una función o operador no toma los argumentos con los tipos que le has pasado. Estos errores suelen ser muy largos y sueltan muchas notas con muchos símbolos, hay que ignorar las notas y centrarse en los errores, hay que priorizar los que salen al principio.

```cpp
#include <iostream>
#include <string>
using namespace std;

class Persona {
  string nombre;
};

int main() {
  Persona persona;
  persona.nombre = "Juan";
  cout << persona << endl;
  return 0;
}
```

``` 
/home/david/org/uni/1/mp/yy/test.cpp:12:8: error: no match for ‘operator<<’ (operand types are ‘std::ostream’ {aka ‘std::basic_ostream<char>’} and ‘Persona’)
   12 |   cout << persona << endl;
      |   ~~~~ ^~ ~~~~
      |   |       |
      |   |       Persona
      |   std::ostream {aka std::basic_ostream<char>}
```
El mensaje de error nos dice que no hay una función operador<< que tome ostream y Persona. 

## Intentar cambiar una variable constante
```cpp
  const Persona persona;
  persona.nombre = "Juan";
```

``` 
/home/david/org/uni/1/mp/yy/test.cpp:11:20: error: no match for ‘operator=’ (operand types are ‘const string’ {aka ‘const std::__cxx11::basic_string<char>’} and ‘const char [5]’)
   11 |   persona.nombre = "Juan";
      |                    ^~~~~~
```
Nos dice que no hay ningún operador = que funcione entre const string (el tipo de persona.nombre) y const char[5], el tipo "Juan". Lo importante aquí es fijarse que cuando describe los tipos de los operandos dice que persona.nombre es const.
