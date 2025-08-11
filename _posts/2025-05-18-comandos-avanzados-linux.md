---
title: "Comandos Avanzados de Linux: Guía Completa para Power Users"
date: 2025-05-19 15:30:00 +0000
categories: [Linux, Terminal]
tags: [linux, comandos-avanzados, awk, sed, grep, find, terminal, shell, bash, text-processing]
---

Si ya dominas los comandos básicos de Linux, es hora de dar el siguiente paso. Esta guía te llevará desde comandos intermedios hasta técnicas avanzadas que usan los administradores de sistemas y especialistas en ciberseguridad.

> 🎯 **Objetivo:** Convertirte en un power user de Linux que puede automatizar tareas complejas, procesar texto masivamente y administrar sistemas como un profesional.

## 🧠 Fundamentos teóricos: ¿Por qué estos comandos son poderosos?

### La filosofía Unix: "Haz una cosa y hazla bien"

Los comandos avanzados de Linux siguen la **filosofía Unix**: cada herramienta tiene un propósito específico, pero pueden **combinarse** para crear soluciones complejas. Esta filosofía se basa en **principios fundamentales**:

1. **Modularidad**: Cada programa hace una tarea específica
2. **Composabilidad**: Los programas se pueden combinar
3. **Transparencia**: Todo es texto cuando es posible
4. **Eficiencia**: Recursos del sistema utilizados óptimamente

#### **1. Pipes (|) - El poder de la composición**

Los pipes son una **abstracción del kernel** que implementa el patrón **productor-consumidor**:

```bash
# Cada comando procesa la salida del anterior
ps aux | grep apache | awk '{print $2}' | xargs kill
```

**¿Cómo funciona internamente?**

El kernel Linux implementa pipes usando **buffers circulares** en memoria:

```c
// Estructura interna simplificada de un pipe
struct pipe_buffer {
    struct page *page;    // Página de memoria
    unsigned int offset;  // Offset en la página
    unsigned int len;     // Longitud de datos
};
```

**Proceso paso a paso:**
1. El shell crea **file descriptors** para cada extremo del pipe
2. Se establece un **buffer de 64KB** en memoria del kernel
3. El proceso escritor **bloquea** si el buffer está lleno
4. El proceso lector **bloquea** si el buffer está vacío
5. El kernel **sincroniza** automáticamente ambos procesos

**Ventajas del diseño:**
- **Concurrencia real**: Ambos procesos ejecutan simultáneamente
- **Gestión automática de memoria**: El kernel maneja el buffering
- **Presión de flujo (backpressure)**: Previene saturación de memoria

#### **2. Redirecciones - Control total de entrada/salida**

Las redirecciones manipulan la **tabla de file descriptors** del proceso:

```bash
comando > archivo     # stdout al archivo (sobrescribe)
comando >> archivo    # stdout al archivo (añade)
comando 2> error.log  # stderr al archivo
comando &> todo.log   # stdout y stderr al archivo
comando < input.txt   # stdin desde archivo
comando 2>&1         # redirige stderr a stdout
```

**File descriptors (FD) - Teoría del SO:**
- **0 (stdin)**: Entrada estándar (por defecto: teclado)
- **1 (stdout)**: Salida estándar (por defecto: terminal)
- **2 (stderr)**: Salida de error (por defecto: terminal)
- **3+**: File descriptors personalizados

**Tabla de file descriptors:**
Cada proceso mantiene una tabla que mapea números a **estructuras file**:

```
FD    |  Archivo/Dispositivo    |  Permisos
------|-------------------------|----------
0     |  /dev/pts/0 (terminal) |  R
1     |  /dev/pts/0 (terminal) |  W
2     |  /dev/pts/0 (terminal) |  W
3     |  /home/user/data.txt   |  R
```

#### **3. Expresiones regulares - Teoría de autómatas y lenguajes formales**

Las regex son un **lenguaje formal** basado en la **teoría de autómatas finitos**:

```bash
# Metacaracteres básicos
.         # Cualquier carácter (excepto \n)
*         # Cero o más del anterior (operador Kleene)
+         # Uno o más del anterior
?         # Cero o uno del anterior
^         # Inicio de línea (ancla)
$         # Final de línea (ancla)
[]        # Clase de caracteres (conjunto)
()        # Grupos de captura (subexpresión)
|         # Alternativa (unión)
```

**Fundamentos matemáticos:**

Las regex pueden describir **lenguajes regulares**, que son una subclase de los lenguajes formales en la **jerarquía de Chomsky**:

```
Lenguajes Regulares (Tipo 3)
    ↑
Lenguajes Libres de Contexto (Tipo 2)
    ↑
Lenguajes Sensibles al Contexto (Tipo 1)
    ↑
Lenguajes Recursivamente Enumerables (Tipo 0)
```

**Motor de regex - Algoritmos de matching:**

1. **NFA (Nondeterministic Finite Automaton)**: 
   - Construcción directa desde regex
   - Múltiples estados activos simultáneamente
   - Backtracking para resolver ambigüedades

2. **DFA (Deterministic Finite Automaton)**:
   - Conversión desde NFA
   - Un solo estado activo
   - Matching en tiempo O(n)

**Ejemplo de construcción NFA:**
Para la regex `a*b`:
```
Estado 0 --ε--> Estado 1 --a--> Estado 1
Estado 1 --b--> Estado 2 (final)
```

#### **4. Procesamiento de streams - Teoría de flujos de datos**

Los comandos Unix procesan datos como **streams infinitos**, no como archivos completos:

**Ventajas del modelo de streams:**
- **Memoria constante**: No necesita cargar archivos completos
- **Procesamiento en tiempo real**: Puede procesar datos conforme llegan
- **Escalabilidad**: Funciona con archivos de cualquier tamaño
- **Composabilidad**: Múltiples filtros en cascada

**Modelo de procesamiento:**
```
Input Stream → Filter 1 → Filter 2 → Filter 3 → Output Stream
```

#### **5. Internals del shell - Parsing y ejecución**

Cuando escribes un comando, el shell realiza estos pasos:

1. **Lexical Analysis**: Divide la línea en tokens
2. **Parsing**: Construye un AST (Abstract Syntax Tree)
3. **Expansion**: Expande variables, globs, etc.
4. **Execution**: Fork/exec de procesos

**Ejemplo de parsing:**
```bash
ls -la | grep "txt" > results.txt
```

Se convierte en:
```
PIPE
├── COMMAND[ls, -la]
└── REDIRECT[>]
    ├── COMMAND[grep, txt]
    └── FILE[results.txt]
```

### 📊 Complejidad computacional de comandos

**Operaciones comunes y su complejidad:**

| Comando | Operación | Complejidad | Memoria |
|---------|-----------|-------------|---------|
| `grep` | Búsqueda lineal | O(n⋅m) | O(1) |
| `sort` | Ordenamiento | O(n log n) | O(n) |
| `uniq` | Eliminación duplicados | O(n) | O(1) |
| `find` | Búsqueda en árbol | O(n) | O(d) |
| `awk` | Procesamiento secuencial | O(n⋅p) | O(r) |

Donde:
- n = tamaño de entrada
- m = tamaño del patrón
- p = complejidad del programa awk
- d = profundidad del directorio
- r = tamaño del registro más grande

---

## 🔍 Maestría en búsqueda: `find` y `locate`

### Teoría: ¿Cómo funciona la búsqueda en filesystems?

#### **Estructura de datos del filesystem**

Linux organiza archivos usando **B-trees** y **hash tables** en la mayoría de filesystems modernos:

**Ext4 filesystem:**
- **Inodes**: Estructuras que contienen metadatos de archivos
- **Directory entries (dentries)**: Mapean nombres a inodes
- **Block groups**: Organizan el espacio en disco
- **Journal**: Garantiza consistencia ante fallos

```c
// Estructura simplificada de inode en ext4
struct ext4_inode {
    __le16  i_mode;        // Permisos y tipo de archivo
    __le16  i_uid;         // User ID del propietario
    __le32  i_size_lo;     // Tamaño del archivo
    __le32  i_atime;       // Tiempo de último acceso
    __le32  i_mtime;       // Tiempo de última modificación
    __le32  i_ctime;       // Tiempo de cambio de inode
    __le32  i_block[15];   // Punteros a bloques de datos
    // ... más campos
};
```

#### **Algoritmos de traversal de directorios**

**Depth-First Search (DFS)**: Utilizado por `find`
```
/home
├── user1/
│   ├── docs/
│   │   └── file1.txt
│   └── file2.txt
└── user2/
    └── file3.txt

Orden DFS: /home → user1 → docs → file1.txt → file2.txt → user2 → file3.txt
```

**Ventajas del DFS:**
- **Memoria constante**: Solo almacena la ruta actual
- **Procesamiento inmediato**: Puede actuar sobre cada archivo encontrado
- **Interrumpible**: Se puede parar en cualquier momento

#### **find vs locate: Análisis de complejidad**

| Aspecto | `find` | `locate` |
|---------|--------|----------|
| **Complejidad temporal** | O(n) donde n = archivos en subtree | O(log m) donde m = archivos en DB |
| **Complejidad espacial** | O(d) donde d = profundidad máxima | O(1) |
| **Preparación** | No requiere | O(n) para crear DB |
| **Actualización** | Tiempo real | Requiere `updatedb` |
| **Flexibilidad** | Criterios complejos | Solo nombre/ruta |

**`find`**: Búsqueda en **tiempo real** con **criterios complejos**
- Utiliza **system calls** como `opendir()`, `readdir()`, `stat()`
- Accede directamente a **metadatos del filesystem**
- Puede realizar **acciones** sobre resultados
- Soporta **predicados complejos** con lógica booleana

**`locate`**: Búsqueda **indexada** ultrarrápida
- Usa base de datos **precomputada** (`/var/lib/mlocate/mlocate.db`)
- Implementa **búsqueda binaria** en índice ordenado
- Actualizada por `updatedb` (típicamente via cron)
- **Limitado a nombres/rutas** de archivos

#### **Implementación de updatedb**

El comando `updatedb` construye el índice usando este algoritmo:

```bash
# Algoritmo simplificado de updatedb
1. Traversal DFS de todos los filesystems montados
2. Para cada archivo/directorio:
   - Verificar permisos de lectura
   - Aplicar filtros de exclusión
   - Añadir ruta al buffer
3. Ordenar rutas lexicográficamente
4. Comprimir y escribir a mlocate.db
```

**Optimizaciones en updatedb:**
- **Pruning**: Evita directorios excluidos
- **Compression**: Usa algoritmos como LZ para reducir tamaño
- **Incremental updates**: Solo procesa cambios desde última ejecución

### El comando `find` - Arquitectura interna

`find` implementa un **evaluador de expresiones** que procesa predicados en forma de árbol:

**Parser de expresiones:**
```bash
find /path -name "*.txt" -size +1M -not -user root
```

Se convierte en un **AST (Abstract Syntax Tree)**:
```
         AND
       /  |  \
   name   size  NOT
   *.txt  +1M    |
              user:root
```

**Evaluación lazy (perezosa):**
- Los predicados se evalúan de **izquierda a derecha**
- **Short-circuit evaluation**: Se para en el primer `false` para AND
- **Optimización automática**: Mueve predicados rápidos al inicio

#### **Predicados de find - Categorías y complejidad**

**Por contenido de inode** (Rápido - O(1)):
```bash
-name, -iname     # Nombre del archivo
-type             # Tipo (f=file, d=directory, l=link)
-size             # Tamaño
-user, -group     # Propietario
-perm             # Permisos
-newer, -older    # Comparación de tiempo
```

**Por metadatos del filesystem** (Medio - O(1) pero más syscalls):
```bash
-atime, -mtime, -ctime    # Tiempos de acceso/modificación
-amin, -mmin, -cmin       # Tiempos en minutos
-empty                    # Archivos/directorios vacíos
-executable, -readable    # Permisos específicos
```

**Por contenido del archivo** (Lento - O(contenido)):
```bash
-exec grep "pattern" {} \;    # Ejecutar comando en cada archivo
-exec test -x {} \;           # Test personalizado
```

#### **Optimización de búsquedas con find**

**1. Orden de predicados:**
```bash
# MALO: Predicado lento primero
find /var -exec grep -l "error" {} \; -name "*.log"

# BUENO: Predicado rápido primero
find /var -name "*.log" -exec grep -l "error" {} \;
```

**2. Limitar scope:**
```bash
# MALO: Busca en todo el sistema
find / -name "config.txt"

# BUENO: Busca solo donde es probable
find /etc /home/user/.config -name "config.txt"
```

**3. Usar -prune para excluir directorios:**
```bash
# Evita buscar en /proc y /sys (muy lentos)
find / \( -path /proc -o -path /sys \) -prune -o -name "*.conf" -print
```

### El comando `find` - Tu mejor amigo para búsquedas

`find` es posiblemente el comando más potente para buscar archivos y directorios en Linux. Su sintaxis completa es:

```bash
find [punto_inicio] [criterios] [acciones]
```

#### Cómo funciona internamente `find`
1. **Traversal**: Recorre directorios recursivamente usando llamadas al sistema `opendir()`, `readdir()`
2. **Evaluación**: Para cada archivo, evalúa los criterios especificados
3. **Acción**: Si todos los criterios se cumplen, ejecuta la acción (por defecto: `-print`)

#### Búsquedas básicas mejoradas
```bash
# Buscar archivos por nombre (case-insensitive)
find /home -iname "*.txt"

# Buscar archivos modificados en los últimos 7 días
find /var/log -mtime -7

# Buscar archivos mayores a 100MB
find /home -size +100M

# Buscar archivos vacíos
find /tmp -empty

# Buscar archivos con permisos específicos
find /etc -perm 644
```

#### Búsquedas por tipo de archivo
```bash
# Solo directorios
find /usr -type d -name "*lib*"

# Solo archivos regulares
find /home -type f -name "*.log"

# Enlaces simbólicos
find /usr/bin -type l

# Archivos ejecutables
find /usr/bin -type f -executable
```

#### Búsquedas por tiempo avanzadas

**Entendiendo los timestamps en Linux:**

- **mtime (modification time)**: Contenido del archivo modificado
- **atime (access time)**: Archivo leído/accedido
- **ctime (change time)**: Metadatos (permisos, propietario) modificados

```bash
# Archivos modificados hace exactamente 30 días
find /backup -mtime 30

# Archivos accedidos en la última hora
find /tmp -amin -60

# Archivos cambiados entre 2 y 5 días atrás
find /var -ctime +2 -ctime -5

# Archivos más nuevos que un archivo específico
find /home -newer /etc/passwd
```

**Sintaxis de tiempo:**
- **+n**: Más de n unidades
- **-n**: Menos de n unidades  
- **n**: Exactamente n unidades

#### Ejecutar acciones en los resultados

**Teoría de las acciones:** `find` puede ejecutar comandos en los archivos encontrados usando:

- **`-exec`**: Ejecuta comando para cada archivo
- **`-ok`**: Como `-exec` pero pide confirmación
- **`{}`**: Placeholder para el nombre del archivo
- **`\;`**: Termina el comando
- **`+`**: Agrupa múltiples archivos en una sola ejecución

```bash
# Ejecutar comando en cada archivo encontrado
find /tmp -name "*.tmp" -exec rm {} \;

# Confirmar antes de ejecutar acción
find /home -name "*.bak" -ok rm {} \;

# Ejecutar comando complejo con múltiples archivos
find /var/log -name "*.log" -exec grep -l "ERROR" {} \;

# Usar xargs para mayor eficiencia (evita crear muchos procesos)
find /home -name "*.txt" -print0 | xargs -0 grep -l "password"
```

**¿Por qué usar `xargs`?**
- **Limitación de argumentos**: Los sistemas tienen límite de argumentos por comando
- **Eficiencia**: `xargs` agrupa archivos para minimizar llamadas al sistema
- **Control de paralelismo**: Puede ejecutar comandos en paralelo

#### Búsquedas complejas con operadores lógicos

**Operadores lógicos en find:**
- **`-a` o `-and`**: Y lógico (por defecto)
- **`-o` o `-or`**: O lógico
- **`!` o `-not`**: NO lógico
- **`(` `)`**: Agrupación (necesita escape: `\(` `\)`)

```bash
# Archivos .log O .txt
find /var -name "*.log" -o -name "*.txt"

# Archivos .conf Y modificados hoy
find /etc -name "*.conf" -a -mtime 0

# Archivos grandes PERO NO en /proc
find / -size +1G -not -path "/proc/*"

# Archivos de usuario específico y con permisos de escritura
find /home -user juan -perm -200
```

### Comando `locate` - Búsqueda ultrarrápida

**¿Cómo funciona `locate`?**

1. **Indexación**: `updatedb` escanea todo el filesystem y crea una base de datos comprimida
2. **Búsqueda**: `locate` busca en esta base de datos usando **algoritmos de búsqueda rápida**
3. **Filtros**: Aplica filtros de seguridad basados en permisos del usuario

```bash
# Actualizar base de datos de locate
sudo updatedb

# Búsqueda rápida por nombre
locate passwd

# Búsqueda case-insensitive
locate -i PASSWORD

# Limitar número de resultados
locate -l 10 "*.conf"

# Buscar solo archivos existentes (verificar que no fueron eliminados)
locate -e httpd.conf
```

**Ventajas y desventajas:**

✅ **Ventajas:**
- Extremadamente rápido (búsqueda en microsegundos)
- No consume recursos del sistema durante la búsqueda
- Ideal para búsquedas frecuentes

❌ **Desventajas:**
- Base de datos puede estar desactualizada
- No puede buscar por metadatos (tamaño, permisos, fecha)
- Limitado a búsquedas por nombre/ruta

---

## 🔤 Procesamiento de texto: `grep`, `awk` y `sed`

### Teoría: Algoritmos de búsqueda de patrones

#### **String matching algorithms - Fundamentos**

La búsqueda de patrones en texto es un problema fundamental en ciencias de la computación. Los diferentes algoritmos tienen distintas complejidades y casos de uso:

**1. Naive Algorithm (Fuerza bruta)**
```c
// Complejidad: O(n*m) donde n = texto, m = patrón
for (int i = 0; i <= n - m; i++) {
    for (int j = 0; j < m; j++) {
        if (texto[i+j] != patron[j]) break;
        if (j == m-1) return i; // Encontrado
    }
}
```

**2. Knuth-Morris-Pratt (KMP)**
- **Complejidad**: O(n + m)
- **Idea**: Precalcula información para evitar backtracking
- **Usado en**: `grep` cuando busca patrones fijos

**3. Boyer-Moore Algorithm**
- **Complejidad**: O(n/m) en el mejor caso, O(n*m) en el peor
- **Idea**: Busca de derecha a izquierda, salta caracteres
- **Usado en**: Búsquedas de texto en editores

**4. Rabin-Karp (Rolling hash)**
- **Complejidad**: O(n + m) promedio
- **Idea**: Usa hash para comparaciones rápidas
- **Usado en**: Detección de plagio, búsqueda de múltiples patrones

#### **Regular expression engines - NFA vs DFA**

Los motores de regex implementan diferentes estrategias:

**NFA (Nondeterministic Finite Automaton) - Usado por PCRE, Perl, Python:**
```
Ventajas:
- Soporte completo de features (backreferences, lookahead)
- Construcción directa desde regex
- Memoria eficiente

Desventajas:
- Backtracking puede ser exponencial O(2^n)
- Vulnerable a "catastrophic backtracking"
```

**DFA (Deterministic Finite Automaton) - Usado por grep, awk tradicional:**
```
Ventajas:
- Tiempo de ejecución garantizado O(n)
- No hay backtracking
- Predictible en rendimiento

Desventajas:
- Features limitadas (no backreferences)
- Conversión NFA→DFA puede explotar en memoria
```

**Ejemplo de catastrophic backtracking:**
```bash
# Esta regex puede tardar exponencialmente
echo "aaaaaaaaaaaaaaaaaaaaaaaX" | grep -E "(a+)+" 
```

#### **Implementación de grep - Arquitectura interna**

GNU grep usa múltiples estrategias según el patrón:

**1. Boyer-Moore** para patrones fijos:
```bash
grep "error" file.txt  # Usa Boyer-Moore
```

**2. DFA** para regex simples:
```bash
grep "^[0-9]+$" file.txt  # Construye DFA
```

**3. NFA** para regex complejas:
```bash
grep "(foo|bar)\1" file.txt  # Requiere backreferences
```

**Optimizaciones en grep:**
- **mmap()**: Mapea archivos completos en memoria virtual
- **SIMD instructions**: Usa SSE/AVX para búsquedas en paralelo
- **Línea-por-línea**: Evita cargar archivos completos
- **Unicode awareness**: Manejo correcto de encodings

### `grep` - Búsqueda de patrones avanzada

#### **Arquitectura y rendimiento**

GNU grep implementa varias optimizaciones:

**1. Fast path para patrones literales:**
```bash
# Optimización especial para strings fijos
grep "literal_string" file.txt
```

**2. Multi-byte character support:**
```bash
# Manejo correcto de UTF-8, UTF-16
grep "café" archivo_unicode.txt
```

**3. Memory mapping para archivos grandes:**
```bash
# grep mapea archivos >64KB usando mmap()
grep "pattern" archivo_10GB.log
```

#### Opciones esenciales
```bash
# Búsqueda case-insensitive
grep -i "error" /var/log/syslog

# Mostrar número de línea
grep -n "root" /etc/passwd

# Mostrar contexto (líneas antes y después)
grep -C 3 "failed" /var/log/auth.log

# Búsqueda recursiva en directorios
grep -r "TODO" /home/user/proyecto/

# Contar coincidencias
grep -c "WARNING" /var/log/messages

# Mostrar solo nombres de archivo con coincidencias
grep -l "database" /etc/*.conf

# Mostrar líneas que NO coinciden
grep -v "^#" /etc/ssh/sshd_config
```

#### Expresiones regulares con grep
```bash
# Buscar líneas que empiecen con "root"
grep "^root" /etc/passwd

# Líneas que terminen con "bash"
grep "bash$" /etc/passwd

# Buscar direcciones IP
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /var/log/access.log

# Buscar emails
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" archivo.txt

# Buscar palabras completas
grep -w "root" /etc/passwd

# Buscar múltiples patrones
grep -E "(error|warning|critical)" /var/log/syslog
```

#### Combinaciones poderosas
```bash
# Buscar en archivos comprimidos
zgrep "error" /var/log/*.gz

# Buscar procesos específicos
ps aux | grep -v grep | grep apache

# Buscar IPs únicas en logs
grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log | sort -u

# Filtrar comentarios y líneas vacías
grep -v "^$" /etc/config | grep -v "^#"
```

### `awk` - Procesamiento de texto programático

#### **Teoría: AWK como lenguaje de programación**

AWK es un **lenguaje de programación especializado** en procesamiento de texto estructurado, basado en el paradigma **pattern-action**:

```
pattern { action }
```

**Modelo de ejecución de AWK:**
1. **Parsing**: Analiza el programa awk y construye un AST
2. **Setup**: Ejecuta bloques `BEGIN`
3. **Main loop**: Para cada línea de entrada:
   - Divide en campos usando FS (Field Separator)
   - Evalúa patrones
   - Ejecuta acciones correspondientes
4. **Cleanup**: Ejecuta bloques `END`

#### **Arquitectura interna del intérprete AWK**

**1. Lexical Analysis (Tokenización):**
```c
// Tokens principales en AWK
enum tokens {
    FIELD,      // $1, $2, $NF
    VARIABLE,   // var, NR, NF
    STRING,     // "texto"
    NUMBER,     // 123, 3.14
    OPERATOR,   // +, -, *, /, %
    PATTERN,    // /regex/
    // ...
};
```

**2. Sintactic Analysis (Parsing):**
El parser construye un AST que representa la estructura del programa:
```
Program
├── Rule 1: Pattern → Action
├── Rule 2: Pattern → Action
└── Rule N: Pattern → Action
```

**3. Runtime Environment:**
```c
// Estado del intérprete AWK simplificado
struct awk_state {
    char **fields;        // $0, $1, $2, ..., $NF
    int NF;              // Número de campos
    int NR;              // Número de registro actual
    char *FS;            // Field separator
    char *RS;            // Record separator
    hash_table variables; // Variables definidas por usuario
    hash_table arrays;   // Arrays asociativos
};
```

#### **Paradigma pattern-action y evaluación**

**Tipos de patrones:**
```bash
# 1. Expresión booleana
$3 > 100 { print $1 }

# 2. Regex
/error/ { count++ }

# 3. Rango de líneas
NR==10, NR==20 { print }

# 4. BEGIN/END (especiales)
BEGIN { FS=":" }
END { print "Total:", count }

# 5. Sin patrón (equivale a 1, siempre true)
{ print NF }
```

**Evaluación de patrones:**
- Los patrones se evalúan **secuencialmente**
- Si un patrón es **true**, se ejecuta su acción
- **Múltiples reglas** pueden ejecutarse para la misma línea
- El control fluye a la **siguiente línea** después de procesar todas las reglas

#### **Sistema de tipos y variables en AWK**

AWK tiene un **sistema de tipos dinámico** con coerción automática:

**1. Tipos primitivos:**
```bash
# String
name = "Juan"

# Number (int o float)
age = 25
price = 99.99

# Coerción automática
result = "10" + 20    # result = 30 (number)
text = 10 " items"    # text = "10 items" (string)
```

**2. Arrays asociativos (hash tables):**
```bash
# Declaración implícita
count["error"] = 5
count["warning"] = 2

# Iteración
for (key in count) {
    print key, count[key]
}

# Arrays multidimensionales (simulados)
matrix[1,2] = "value"  # Internamente: "1" SUBSEP "2"
```

**3. Variables built-in críticas:**

| Variable | Propósito | Ejemplo |
|----------|-----------|---------|
| `$0` | Línea completa | "nombre:password:uid:gid" |
| `$1, $2, ...` | Campos individuales | $1="nombre", $2="password" |
| `NF` | Número de campos | 4 |
| `NR` | Número de línea actual | 1, 2, 3, ... |
| `FS` | Separador de campos | ":", "\t", " " |
| `RS` | Separador de registros | "\n", "\0" |
| `OFS` | Sep. salida campos | " " |
| `ORS` | Sep. salida registros | "\n" |

#### **Algoritmo de splitting de campos**

Cuando AWK procesa una línea, implementa este algoritmo:

```c
// Algoritmo simplificado de field splitting
void split_fields(char *line, char *fs) {
    if (fs is regex) {
        // Usa regcomp()/regexec() para encontrar separadores
        split_by_regex(line, fs);
    } else if (fs == " ") {
        // Caso especial: split por whitespace
        skip_leading_whitespace(line);
        split_by_whitespace(line);
    } else if (strlen(fs) == 1) {
        // Carácter único: optimización
        split_by_char(line, fs[0]);
    } else {
        // String multi-carácter
        split_by_string(line, fs);
    }
}
```

**Complejidad de operaciones:**
- **Field access ($1, $2, ...)**: O(1) después del splitting inicial
- **Field splitting**: O(n) donde n = longitud de línea
- **Array access**: O(1) promedio (hash table)
- **Pattern matching**: O(m) donde m = longitud del patrón

#### **Memory management y eficiencia**

**Estrategias de AWK para manejar memoria:**

1. **Lazy field splitting**: Solo divide campos cuando se acceden
2. **String interning**: Reutiliza strings comunes
3. **Garbage collection**: Libera variables no utilizadas
4. **Buffer recycling**: Reutiliza buffers para líneas

`awk` es un lenguaje de programación completo para procesamiento de texto estructurado.

#### Conceptos básicos de awk
```bash
# Imprimir columnas específicas
awk '{print $1, $3}' /etc/passwd

# Imprimir líneas con más de 80 caracteres
awk 'length > 80' archivo.txt

# Procesar solo líneas que coincidan con patrón
awk '/error/ {print $0}' /var/log/syslog

# Imprimir número de línea y contenido
awk '{print NR ": " $0}' archivo.txt
```

#### Variables internas útiles
```bash
# NF = Número de campos, NR = Número de línea
awk '{print "Línea " NR " tiene " NF " campos"}' datos.txt

# FS = Separador de campo
awk 'BEGIN {FS=":"} {print $1, $7}' /etc/passwd

# Longitud de cada línea
awk '{print length($0)}' archivo.txt

# Imprimir último campo de cada línea
awk '{print $NF}' datos.txt
```

#### Operaciones matemáticas y estadísticas
```bash
# Sumar valores de la tercera columna
awk '{sum += $3} END {print "Total:", sum}' numeros.txt

# Calcular promedio
awk '{sum += $1; count++} END {print "Promedio:", sum/count}' datos.txt

# Encontrar máximo valor
awk '{if($1 > max) max = $1} END {print "Máximo:", max}' numeros.txt

# Contar líneas por categoría
awk '{count[$2]++} END {for(i in count) print i, count[i]}' datos.txt
```

#### Scripts awk complejos
```bash
# Procesar log de Apache
awk '{
    ip = $1
    size = $10
    total_bytes += size
    requests[ip]++
} 
END {
    print "Total bytes:", total_bytes
    print "Requests por IP:"
    for(ip in requests) print ip, requests[ip]
}' access.log

# Analizar uso de CPU
ps aux | awk '
NR > 1 {
    cpu += $3
    mem += $4
    processes++
}
END {
    print "Procesos:", processes
    print "CPU promedio:", cpu/processes "%"
    print "Memoria promedio:", mem/processes "%"
}'
```

### `sed` - Editor de flujo para transformación de texto

#### **Teoría: Stream editors y automatas finitos**

`sed` (Stream EDitor) es un **editor de flujo no interactivo** que procesa texto línea por línea usando un **autómata finito**. Su diseño se basa en tres componentes principales:

**1. Pattern Space (Espacio de patrón):**
- Buffer principal donde se carga la línea actual
- Todas las operaciones se realizan aquí
- Se imprime automáticamente al final (salvo `-n`)

**2. Hold Space (Espacio de retención):**
- Buffer auxiliar para almacenamiento temporal
- Permite operaciones complejas entre líneas
- Persistente durante toda la ejecución

**3. Máquina de estados:**
```
Input → Pattern Space → Comandos → Output
         ↕
    Hold Space
```

#### **Modelo de ejecución de sed**

**Ciclo principal de sed:**
```c
// Algoritmo simplificado del motor sed
while ((line = read_next_line()) != NULL) {
    load_pattern_space(line);
    
    for (each_command in script) {
        if (address_matches(command.address, current_line)) {
            execute_command(command);
            if (command.flags & NEXT_CYCLE) break;
        }
    }
    
    if (!quiet_mode) {
        print_pattern_space();
    }
    clear_pattern_space();
}
```

**Addressing scheme (Sistema de direcciones):**
```bash
# 1. Número de línea
5s/old/new/           # Solo línea 5

# 2. Rango de líneas
5,10s/old/new/        # Líneas 5 a 10

# 3. Regex
/pattern/s/old/new/   # Líneas que coincidan con pattern

# 4. Último registro
$s/old/new/           # Solo última línea

# 5. Rango regex
/start/,/end/s/old/new/  # Desde 'start' hasta 'end'

# 6. Inverso
5!s/old/new/          # Todas las líneas EXCEPTO la 5
```

#### **Arquitectura de comandos sed**

**Categorías de comandos por función:**

**1. Comandos de sustitución (s):**
```bash
# Sintaxis completa: s/regex/replacement/flags
s/old/new/g           # g = global (todas las ocurrencias)
s/old/new/2           # Solo segunda ocurrencia
s/old/new/p           # p = print (imprimir líneas cambiadas)
s/old/new/w file      # w = write (escribir líneas cambiadas a archivo)
```

**Implementación interna de sustitución:**
```c
// Proceso simplificado de s///
1. Compilar regex del patrón
2. Buscar primera coincidencia en pattern space
3. Si encuentra:
   - Expandir replacement (& y \1, \2, etc.)
   - Reemplazar en pattern space
   - Si flag 'g': repetir desde paso 2
4. Aplicar flags (p, w, etc.)
```

**2. Comandos de flujo de control:**
```bash
n     # Lee siguiente línea a pattern space
N     # Añade siguiente línea a pattern space
d     # Elimina pattern space, salta a siguiente ciclo
D     # Elimina hasta primer \n en pattern space
h     # Copia pattern space a hold space
H     # Añade pattern space a hold space
g     # Copia hold space a pattern space
G     # Añade hold space a pattern space
x     # Intercambia pattern space y hold space
```

**3. Comandos de inserción/eliminación:**
```bash
a\    # Append (añadir después)
i\    # Insert (insertar antes)
c\    # Change (cambiar línea completa)
d     # Delete (eliminar)
```

#### **Expresiones regulares en sed**

`sed` usa **Basic Regular Expressions (BRE)** por defecto:

**Diferencias BRE vs ERE:**
```bash
# BRE (sed default)
\+      # Uno o más (necesita escape)
\?      # Cero o uno (necesita escape)
\|      # Alternativa (necesita escape)
\( \)   # Grupos (necesitan escape)

# ERE (sed -E)
+       # Uno o más (sin escape)
?       # Cero o uno (sin escape)
|       # Alternativa (sin escape)
( )     # Grupos (sin escape)
```

**Backreferences en sed:**
```bash
# Capturar grupos con \( \)
sed 's/\([0-9]*\)\.\([0-9]*\)/\2.\1/'  # Intercambia 123.456 → 456.123

# & representa la coincidencia completa
sed 's/error/[ERROR: &]/'  # error → [ERROR: error]
```

#### **Pattern space vs Hold space - Casos de uso**

**Ejemplo: Intercambiar líneas pares e impares**
```bash
# Usa hold space para "recordar" línea anterior
sed 'N;s/\(.*\)\n\(.*\)/\2\n\1/' archivo.txt
```

**Ejemplo: Eliminar líneas duplicadas consecutivas**
```bash
# Compara pattern space con hold space
sed '$!N; /^\(.*\)\n\1$/!P; D'
```

**Análisis del comando:**
1. `$!N`: Si no es última línea, lee siguiente línea
2. `/^\(.*\)\n\1$/!P`: Si las líneas NO son iguales, imprime primera
3. `D`: Elimina primera línea del pattern space

#### **Optimización y rendimiento de sed**

**Factores de rendimiento:**
1. **Compilación de regex**: Se hace una vez por comando
2. **Direccionamiento**: Usar rangos específicos vs procesar todo
3. **Hold space**: Usar solo cuando sea necesario
4. **Comandos múltiples**: `-e` vs múltiples invocaciones

**Mejores prácticas:**
```bash
# MALO: Múltiples invocaciones
cat file | sed 's/a/b/' | sed 's/c/d/' | sed '/pattern/d'

# BUENO: Script único
sed -e 's/a/b/' -e 's/c/d/' -e '/pattern/d' file

# MEJOR: Archivo de script para casos complejos
sed -f script.sed file
```

**Complejidad computacional:**
- **Sustitución simple**: O(n) donde n = longitud de línea
- **Comandos N/D**: O(k×n) donde k = líneas en pattern space
- **Hold space operations**: O(m) donde m = longitud de hold space

#### Sustituciones básicas
```bash
# Reemplazar primera ocurrencia por línea
sed 's/old/new/' archivo.txt

# Reemplazar todas las ocurrencias
sed 's/old/new/g' archivo.txt

# Reemplazar solo en líneas específicas
sed '5,10s/old/new/g' archivo.txt

# Reemplazar usando delimitador diferente
sed 's|/old/path|/new/path|g' config.txt
```

#### Eliminación y inserción
```bash
# Eliminar líneas que contengan patrón
sed '/pattern/d' archivo.txt

# Eliminar líneas vacías
sed '/^$/d' archivo.txt

# Eliminar líneas 5 a 10
sed '5,10d' archivo.txt

# Insertar texto después de patrón
sed '/pattern/a\Texto a insertar' archivo.txt

# Insertar texto antes de patrón
sed '/pattern/i\Texto a insertar' archivo.txt
```

#### Transformaciones avanzadas
```bash
# Cambiar a mayúsculas/minúsculas
sed 's/.*/\U&/' archivo.txt  # Mayúsculas
sed 's/.*/\L&/' archivo.txt  # Minúsculas

# Eliminar espacios al inicio y final
sed 's/^[ \t]*//;s/[ \t]*$//' archivo.txt

# Numerar líneas
sed '=' archivo.txt | sed 'N;s/\n/: /'

# Reemplazar múltiples espacios por uno solo
sed 's/  */ /g' archivo.txt
```

#### Scripts sed complejos
```bash
# Limpiar archivo de configuración
sed -e '/^#/d' -e '/^$/d' -e 's/[ \t]*$//' config.conf

# Procesar archivo CSV
sed 's/,/ | /g; s/^/| /; s/$/ |/' datos.csv

# Convertir texto a HTML
sed -e 's/&/\&amp;/g' -e 's/</\&lt;/g' -e 's/>/\&gt;/g' archivo.txt
```

---

## 🔧 Gestión de procesos y sistema

### `ps` - Información detallada de procesos
```bash
# Ver todos los procesos con información completa
ps aux

# Procesos en formato árbol
ps auxf

# Procesos de usuario específico
ps -u juan

# Procesos con más uso de CPU
ps aux --sort=-%cpu | head -10

# Procesos con más uso de memoria
ps aux --sort=-%mem | head -10

# Información personalizada
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu
```

### `top` y `htop` - Monitoreo en tiempo real
```bash
# Ordenar por diferentes criterios en top
# P - CPU, M - Memoria, T - Tiempo
top -o %CPU

# Mostrar solo procesos de usuario específico
top -u juan

# Actualizar cada 2 segundos
top -d 2

# Modo batch (sin interacción)
top -b -n 1
```

### `lsof` - Lista de archivos abiertos
```bash
# Ver archivos abiertos por proceso específico
lsof -p 1234

# Ver qué procesos usan un archivo
lsof /var/log/syslog

# Ver conexiones de red
lsof -i

# Ver puertos específicos
lsof -i :80
lsof -i :22

# Ver archivos abiertos en directorio
lsof +D /tmp/

# Procesos usando archivos eliminados
lsof | grep deleted
```

### `netstat` y `ss` - Información de red
```bash
# Ver todas las conexiones
netstat -tuln

# Ver conexiones establecidas
netstat -tun | grep ESTABLISHED

# Ver procesos asociados a puertos
netstat -tulpn

# ss - reemplazo moderno de netstat
ss -tuln
ss -tulpn | grep :80
ss -s  # Estadísticas resumidas
```

---

## 📁 Manejo de archivos y compresión

### `tar` - Archivado avanzado
```bash
# Crear archivo tar con compresión
tar -czf backup.tar.gz /home/user/

# Crear con compresión bzip2 (mejor compresión)
tar -cjf backup.tar.bz2 /home/user/

# Crear con compresión xz (máxima compresión)
tar -cJf backup.tar.xz /home/user/

# Extraer mostrando progreso
tar -xzf archivo.tar.gz --verbose

# Extraer archivos específicos
tar -xzf backup.tar.gz ruta/específica/

# Ver contenido sin extraer
tar -tzf archivo.tar.gz

# Añadir archivos a tar existente
tar -rzf archivo.tar archivo_nuevo.txt

# Crear backup con fecha
tar -czf backup_$(date +%Y%m%d).tar.gz /home/user/
```

### `rsync` - Sincronización avanzada
```bash
# Sincronización básica con progreso
rsync -av --progress /origen/ /destino/

# Backup incremental
rsync -av --link-dest=/backup/anterior /datos/ /backup/nuevo/

# Sincronización por red
rsync -av -e ssh /local/ user@server:/remoto/

# Excluir archivos/directorios
rsync -av --exclude='*.tmp' --exclude='cache/' /origen/ /destino/

# Eliminar archivos en destino que no existen en origen
rsync -av --delete /origen/ /destino/

# Modo dry-run (simular sin ejecutar)
rsync -av --dry-run /origen/ /destino/

# Transferir solo archivos modificados
rsync -av --update /origen/ /destino/
```

### Comando `dd` - Copia a bajo nivel
```bash
# Crear imagen de disco
dd if=/dev/sda of=backup.img bs=4M status=progress

# Crear archivo de prueba de tamaño específico
dd if=/dev/zero of=testfile bs=1M count=100

# Benchmark de velocidad de escritura
dd if=/dev/zero of=testfile bs=1M count=1000 oflag=direct

# Copiar solo primeros sectores (MBR)
dd if=/dev/sda of=mbr.img bs=512 count=1

# Crear USB booteable
dd if=linux.iso of=/dev/sdb bs=4M status=progress oflag=sync
```

---

## 🌐 Herramientas de red

### `curl` - Cliente HTTP/HTTPS avanzado
```bash
# Descargar archivo con progreso
curl -O -# http://example.com/file.zip

# Seguir redirecciones
curl -L http://example.com/redirect

# Enviar datos POST
curl -X POST -d "user=juan&pass=123" http://example.com/login

# Enviar JSON
curl -X POST -H "Content-Type: application/json" -d '{"name":"juan"}' http://api.example.com/users

# Autenticación básica
curl -u usuario:password http://example.com/protected

# Guardar cookies y usarlas
curl -c cookies.txt -b cookies.txt http://example.com/

# Ver headers de respuesta
curl -I http://example.com/

# Timeout y reintentos
curl --connect-timeout 10 --max-time 30 --retry 3 http://example.com/
```

### `wget` - Descarga avanzada
```bash
# Descargar sitio completo
wget -r -np -k -p http://example.com/

# Continuar descarga interrumpida
wget -c http://example.com/largefile.zip

# Descargar en background
wget -b http://example.com/file.zip

# Limitar velocidad de descarga
wget --limit-rate=200k http://example.com/file.zip

# Descargar con autenticación
wget --user=usuario --password=password http://example.com/file.zip

# Descargar múltiples archivos de lista
wget -i lista_urls.txt
```

### `nmap` - Escaneo de red (para administradores)
```bash
# Escaneo básico de red local
nmap 192.168.1.0/24

# Escaneo de puertos específicos
nmap -p 22,80,443 192.168.1.100

# Detección de servicios y versiones
nmap -sV 192.168.1.100

# Escaneo sigiloso
nmap -sS 192.168.1.100

# Detectar hosts activos
nmap -sn 192.168.1.0/24
```

---

## 🔒 Seguridad y permisos avanzados

### Gestión de permisos especiales
```bash
# Setuid, setgid, sticky bit
chmod 4755 archivo  # setuid
chmod 2755 directorio  # setgid
chmod 1755 directorio  # sticky bit

# Permisos con notación octal completa
chmod 4755 /usr/bin/sudo  # rwsr-xr-x

# Cambiar propietario recursivamente
chown -R usuario:grupo /directorio/

# Cambiar solo grupo
chgrp -R grupo /directorio/
```

### ACLs (Access Control Lists)
```bash
# Ver ACLs existentes
getfacl archivo.txt

# Establecer ACL para usuario específico
setfacl -m u:juan:rw archivo.txt

# Establecer ACL para grupo
setfacl -m g:developers:rwx directorio/

# ACL por defecto para directorio
setfacl -d -m u:juan:rwx directorio/

# Eliminar ACL
setfacl -x u:juan archivo.txt

# Eliminar todas las ACLs
setfacl -b archivo.txt
```

### `sudo` avanzado
```bash
# Ver permisos sudo del usuario actual
sudo -l

# Ejecutar comando como otro usuario
sudo -u postgres psql

# Mantener variables de entorno
sudo -E comando

# Ejecutar con shell específico
sudo -s

# Ver log de comandos sudo
sudo journalctl -f /var/log/auth.log | grep sudo
```

---

## 🔄 Automatización y scripting

### Programación de tareas con `cron`
```bash
# Editar crontab personal
crontab -e

# Ver crontab actual
crontab -l

# Ejemplos de programación:
# Cada minuto
* * * * * /ruta/script.sh

# Cada día a las 2:30 AM
30 2 * * * /backup/daily.sh

# Cada lunes a las 9:00 AM
0 9 * * 1 /scripts/weekly-report.sh

# Cada 15 minutos
*/15 * * * * /monitoring/check-status.sh

# Variables de entorno en cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
0 2 * * * /backup/script.sh
```

### `at` - Programación única
```bash
# Ejecutar comando en tiempo específico
echo "/backup/script.sh" | at 14:30

# Ejecutar mañana a las 9 AM
echo "echo 'Buenos días'" | at 9am tomorrow

# Ver trabajos programados
atq

# Eliminar trabajo programado
atrm 2
```

### Variables y expansión avanzada
```bash
# Expansión de llaves
echo archivo{1,2,3}.txt  # archivo1.txt archivo2.txt archivo3.txt
cp archivo.txt{,.bak}    # cp archivo.txt archivo.txt.bak

# Expansión de rangos
echo {1..10}             # 1 2 3 4 5 6 7 8 9 10
echo {a..z}              # a b c d ... z
echo {01..10}            # 01 02 03 ... 10

# Sustitución de comandos
fecha=$(date +%Y%m%d)
archivos=$(ls *.txt | wc -l)

# Parámetros por defecto
archivo=${1:-"default.txt"}  # Si $1 está vacío, usar "default.txt"
```

---

## 📊 Monitoreo y análisis del sistema

### `iostat`, `vmstat`, `sar` - Estadísticas del sistema
```bash
# I/O statistics
iostat -x 1    # Actualizar cada segundo
iostat -d 2 5  # Mostrar solo discos, cada 2 seg, 5 veces

# Virtual memory statistics
vmstat 1
vmstat -s      # Estadísticas resumidas

# System Activity Reporter
sar -u 1 10    # CPU usage cada segundo, 10 veces
sar -r 1 10    # Memoria cada segundo
sar -d 1 10    # Disk I/O
```

### `strace` y `ltrace` - Debugging de procesos
```bash
# Rastrear llamadas al sistema
strace -p 1234                    # Proceso específico
strace -e open,read,write comando # Solo ciertas llamadas
strace -f comando                 # Incluir procesos hijo

# Rastrear llamadas a librerías
ltrace comando
ltrace -p 1234
```

### Análisis de logs
```bash
# Ver logs en tiempo real
tail -f /var/log/syslog

# Buscar patrones en múltiples logs
grep -h "ERROR" /var/log/*.log | sort | uniq -c

# Analizar logs de Apache/Nginx
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10

# Estadísticas de errores por hora
grep "ERROR" /var/log/app.log | awk '{print $1" "$2}' | cut -d: -f1-2 | sort | uniq -c
```

---

## 🎯 Casos de uso prácticos para administradores

### Script de backup inteligente
```bash
#!/bin/bash
# Backup incremental con rotación

BACKUP_DIR="/backup"
SOURCE_DIR="/home"
DATE=$(date +%Y%m%d_%H%M%S)
LAST_BACKUP=$(find $BACKUP_DIR -maxdepth 1 -type d -name "backup_*" | sort | tail -1)

# Crear backup incremental
if [[ -n "$LAST_BACKUP" ]]; then
    rsync -av --link-dest="$LAST_BACKUP" "$SOURCE_DIR/" "$BACKUP_DIR/backup_$DATE/"
else
    rsync -av "$SOURCE_DIR/" "$BACKUP_DIR/backup_$DATE/"
fi

# Eliminar backups antiguos (mantener solo 7)
ls -t "$BACKUP_DIR"/backup_* | tail -n +8 | xargs rm -rf

echo "Backup completado: $BACKUP_DIR/backup_$DATE"
```

### Monitoreo de recursos
```bash
#!/bin/bash
# Script de monitoreo de sistema

LOG_FILE="/var/log/system-monitor.log"

# Función para logging
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

# Verificar uso de CPU
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
if (( $(echo "$CPU_USAGE > 80" | bc -l) )); then
    log_message "ALERTA: CPU uso alto: ${CPU_USAGE}%"
fi

# Verificar uso de memoria
MEM_USAGE=$(free | grep Mem | awk '{printf "%.1f", $3/$2 * 100.0}')
if (( $(echo "$MEM_USAGE > 90" | bc -l) )); then
    log_message "ALERTA: Memoria uso alto: ${MEM_USAGE}%"
fi

# Verificar espacio en disco
df -h | awk 'NR>1 {
    usage = substr($5, 1, length($5)-1)
    if (usage > 85) 
        print "[" strftime("%Y-%m-%d %H:%M:%S") "] ALERTA: Disco " $6 " uso alto: " usage "%"
}' >> "$LOG_FILE"
```

### Análisis de logs automatizado
```bash
#!/bin/bash
# Análisis automático de logs de seguridad

AUTH_LOG="/var/log/auth.log"
REPORT_FILE="/tmp/security_report_$(date +%Y%m%d).txt"

echo "=== REPORTE DE SEGURIDAD ===" > "$REPORT_FILE"
echo "Fecha: $(date)" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Intentos de login fallidos
echo "=== INTENTOS DE LOGIN FALLIDOS ===" >> "$REPORT_FILE"
grep "Failed password" "$AUTH_LOG" | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head -10 >> "$REPORT_FILE"

# IPs sospechosas (más de 10 intentos fallidos)
echo -e "\n=== IPs SOSPECHOSAS ===" >> "$REPORT_FILE"
grep "Failed password" "$AUTH_LOG" | awk '{print $(NF-3)}' | sort | uniq -c | awk '$1 > 10 {print $2 " (" $1 " intentos)"}' >> "$REPORT_FILE"

# Logins exitosos de root
echo -e "\n=== LOGINS ROOT EXITOSOS ===" >> "$REPORT_FILE"
grep "Accepted.*root" "$AUTH_LOG" | tail -10 >> "$REPORT_FILE"

echo "Reporte generado: $REPORT_FILE"
```

---

## 🚀 Consejos para convertirte en un Power User

### 1. **Combina comandos con pipes**
```bash
# Pipeline complejo de análisis
ps aux | awk '{print $11}' | sort | uniq -c | sort -nr | head -10

# Análisis de uso de red
netstat -tuln | grep LISTEN | awk '{print $4}' | cut -d: -f2 | sort -n
```

### 2. **Usa aliases inteligentes**
```bash
# Agregar a ~/.bashrc
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias grep='grep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
alias h='history'
alias c='clear'
alias df='df -h'
alias du='du -h'
alias free='free -h'
alias ps='ps auxf'
alias mkdir='mkdir -pv'

# Aliases para desarrollo
alias gst='git status'
alias gco='git checkout'
alias gad='git add .'
alias gcm='git commit -m'
```

### 3. **Funciones bash útiles**
```bash
# Función para extraer cualquier archivo comprimido
extract() {
    if [ -f $1 ]; then
        case $1 in
            *.tar.bz2)   tar xjf $1     ;;
            *.tar.gz)    tar xzf $1     ;;
            *.bz2)       bunzip2 $1     ;;
            *.rar)       unrar e $1     ;;
            *.gz)        gunzip $1      ;;
            *.tar)       tar xf $1      ;;
            *.tbz2)      tar xjf $1     ;;
            *.tgz)       tar xzf $1     ;;
            *.zip)       unzip $1       ;;
            *.Z)         uncompress $1  ;;
            *.7z)        7z x $1        ;;
            *)           echo "'$1' no se puede extraer" ;;
        esac
    else
        echo "'$1' no es un archivo válido"
    fi
}

# Función para crear directorio y entrar en él
mkcd() {
    mkdir -p "$1" && cd "$1"
}
```

### 4. **Configuración de vim básica**
```bash
# ~/.vimrc básico
set number
set hlsearch
set ignorecase
set smartcase
set autoindent
set expandtab
set tabstop=4
set shiftwidth=4
syntax enable
```

---

## 📚 Recursos para seguir aprendando

### Documentación esencial
```bash
# Los mejores recursos en tu sistema
man comando              # Manual completo
info comando             # Documentación GNU
comando --help           # Ayuda rápida
/usr/share/doc/          # Documentación adicional
```

### Libros recomendados
- **"The Linux Command Line"** - William Shotts
- **"Unix and Linux System Administration Handbook"** - Nemeth, Snyder, et al.
- **"Learning the bash Shell"** - Cameron Newham

### Práctica continua
1. **Configura un laboratorio** con VMs o contenedores
2. **Automatiza tareas diarias** con scripts
3. **Participa en comunidades** como r/linux, Stack Overflow
4. **Practica en CTFs** para ciberseguridad
5. **Contribuye a proyectos** open source

---

## 🎯 Próximos pasos

Ahora que dominas estos comandos avanzados:

1. **Practica regularmente** - La maestría viene con la repetición
2. **Combina herramientas** - Los comandos más poderosos vienen de combinaciones
3. **Automatiza todo** - Si lo haces más de 2 veces, automatízalo
4. **Aprende scripting** - Bash, Python, o tu lenguaje preferido
5. **Explora herramientas específicas** - Para tu área (dev, sysadmin, seguridad)

💡 **Recuerda:** Un verdadero power user no memoriza todos los comandos, sino que sabe dónde encontrar la información y cómo combinar herramientas para resolver problemas complejos.

---

**Andrés Núñez**
