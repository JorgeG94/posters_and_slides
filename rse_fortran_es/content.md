## Title: Fortran a sus casi 70 anios: portabilidad, GPUs, y una historia de exito

Presenter: Jorge Luis Galvez Vallejo (NCI)

Logo NCI

Audiencia: conferencia de RSE (ingenieria de software de investigacion). NO son estudiantes.
Se puede hablar de CI, tests, empaquetado y compiladores sin pedir permiso.
15 min -> ~13 min hablando, ~10 laminas.

Diferencia con la charla en ingles: alla se presenta el ARGUMENTO (la pila pic, el hueco
de RSE). Aca se presenta el CASO DE ESTUDIO (Rakali a fondo). Misma tesis, vehiculo distinto.

---

## 1. El lenguaje de alto nivel mas viejo del mundo  [0:30 -> 2:00]

Linea de tiempo, una lamina:

- 1957 Fortran
- 1990 arrays como ciudadanos de primera clase
- 2003 programacion orientada a objetos
- 2008 `do concurrent` (memoria compartida) y coarrays (memoria distribuida)
- 2018 / 2023 interoperabilidad con C, mejoras a do concurrent

Frase: no es un lenguaje viejo, es un lenguaje con historia. Casi todo lo que la gente
critica de Fortran se decidio antes de 1990.

---

## 2. La mala fama, honestamente  [2:00 -> 3:30]

No defender lo indefendible. Empezar por los problemas reales:

- Compiladores: comportamiento distinto entre GNU, Intel, NVIDIA, Cray, AMD, Flang
- Compatibilidad con versiones anteriores: el codigo de 1980 todavia compila, y eso tiene
  un costo -- el lenguaje carga con todo
- Cada vez menos usuarios
- La docencia se fue: metodos numericos y fisica computacional ahora se ensenian con
  Python, C++, Julia. Fortran perdio la puerta de entrada

C y C++ se volvieron la alternativa por defecto para el calculo cientifico moderno.

---

## 3. Y sin embargo  [3:30 -> 4:30]

Lo que Fortran ofrece hoy, sin propaganda:

- Arrays de primera clase: rebanadas, elemental, reducciones, sin escribir el bucle
- Orientacion a objetos
- Memoria dinamica (casi) segura: allocatable, move_alloc
- Paralelismo nativo en el estandar

El punto central de la charla: Fortran permite que un cientifico escriba codigo rapido
sin volverse primero programador de sistemas.

---

## 4. Un solo bucle, cualquier dispositivo  [4:30 -> 6:00]

```fortran
do concurrent(k=1:nk, j=1:nj, i=1:ni)
  c(i,j,k) = alpha * a(i,j,k) + b(i,j,k)
end do
```

contra el muro de directivas:

```fortran
!$omp target teams distribute parallel do collapse(3)
!$acc parallel loop collapse(3)
```

Esto es Fortran estandar. Sin dialecto de pragmas, sin vendor, sin segundo lenguaje.
La bandera del compilador decide serial / multicore / GPU.

---

## 5. Coarrays: la promesa y la realidad  [6:00 -> 7:00]

El abstract los promete, hay que hablar de ellos. Con honestidad:

- F2008 los introdujo, el disenio es elegante
- Pero la implementacion depende del compilador: GCC necesita OpenCoarrays, Intel trae el
  suyo, Cray el suyo es bueno, NVIDIA y AMD practicamente no existen
- Mi criterio es **portable y listo para produccion, o no cuenta**. do concurrent lo pasa.
  Los coarrays todavia no
- Para memoria distribuida uso MPI. Por eso existe pic-mpi
- Podria anadir un backend de coarrays a pic-mpi. Decidi que no: la mayoria de los runtimes
  de coarrays estan implementados sobre MPI, asi que estaria envolviendo una envoltura

Ante un publico de RSE esta lamina compra credibilidad para todo lo demas.

---

## 6. Rakali -- EL CENTRO DE LA CHARLA  [7:00 -> 10:30]

Aqui se gastan 3 minutos y medio. Es el caso de estudio.

Que es:
- Simulacion hidrodinamica: costas, rios, inundaciones, oceano abierto
- 100% Fortran

Como esta hecho (esto es lo que le interesa a un publico de RSE, no los mapas bonitos):
- Portabilidad real: GPUs NVIDIA / AMD / Intel. CPUs Apple / ARM / Intel / AMD
- Un solo codigo fuente. Sin ramas por vendor, sin kernels duplicados
- Tabla de plataformas -- esta es la lamina que la gente fotografia

Figuras disponibles en acomo_rakali/imgs:
- fig_setting.png (el problema)
- fig_speedup.png / fig_performance.png / fig_strong_scaling.png (los numeros)
- le_compound.png / fig_slr_surge.png (la ciencia que habilita)

Frase: no es un experimento. Es codigo que produce ciencia, en siete arquitecturas.

---

## 7. Lo que costo: el ecosistema no es portable  [10:30 -> 11:45]

El giro de la charla. Rakali funciona, pero mantenerlo revelo el problema real:

- stdlib es muy buena y en la practica compila con GNU
- `mpi_f08` necesita vapaa (Hammond) para cruzar compiladores
- Cada proyecto reimplementa las mismas cinco cosas: ordenamiento, logging, timers, strings

El lenguaje es portable. El ecosistema no. Y ese hueco es de ingenieria, no de ciencia.

---

## 8. La pila pic  [11:45 -> 12:30]

Un diagrama, 45 segundos, no abrir ninguno:

- **pic** -- rutinas nivel stdlib en Fortran nativo
- **pic-blas**, **pic-mpi**, **pic-device**
- Interoperabilidad: **libfint** (libcint portado a Fortran), interfaz a cuEST

Y como prueba de que la pila sostiene aplicaciones reales en dominios distintos:
- **metalquicha** -- quimica cuantica, tres motores tras una sola interfaz
- **terco** -- integrales de repulsion electronica en GPU solo con do concurrent
  (una sola V100 de 2017, 3.5x mas rapido que Q-Chem con 104 hilos en Sapphire Rapids)

Mencionar, no explicar. Hidrodinamica y quimica cuantica sobre la misma base: eso hace que
pic sea infraestructura y no una libreria de conveniencia.

---

## 9. Cierre  [12:30 -> 13:00]

Si ignoramos la propaganda negativa, hay un lenguaje muy poderoso al alcance de cualquiera.
Pero no se sostiene solo. Necesita CI, empaquetado, tests, disenio de APIs.
Fortran no necesita reescribirse. Necesita ingenieria. Ese trabajo es de RSE y esta libre.

Enlaces: pic, pic-mpi, pic-blas, pic-device, rakali, metalquicha, terco, libfint, fortran-lang

---

## Laminas de respaldo

- terco a detalle (batches de shell pairs, prior art Alkan / del Angel)
- MOM6 como tercer testigo, por si preguntan por codigo legado grande
- metalquicha: testing diferencial contra PySCF
- Detalle de compiladores probados
