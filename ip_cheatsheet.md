# Tarjeta de referencia: máscaras, wildcards y mnemotecnias

## Regla básica
- Fórmula: total direcciones = 2^(32 - prefijo).
  - Ej.: `/27` → 2^(32-27) = 2^5 = 32 direcciones.
- Wildcard = máscara invertida (bitwise NOT).
- Tamaño de bloque = 256 - (valor de la máscara en el octeto que cambia).

## Mnemotecnias rápidas
  - Frase: "/24 = 256, sube uno, corta a la mitad".
  - Esto da los incrementos de red: `.0, .32, .64, .96, ...` en 3er/4º octeto según corresponda.

## Tabla compacta (prefijos más usados)
Prefijo | Máscara | Wildcard | Direcciones totales | Hosts usables (trad.)
-|-|-|-|-
`/32` | `255.255.255.255` | `0.0.0.0` | 1 | 0
`/31` | `255.255.255.254` | `0.0.0.1` | 2 | 0 (RFC3021: 2 para enlaces P2P)
`/30` | `255.255.255.252` | `0.0.0.3` | 4 | 2
`/29` | `255.255.255.248` | `0.0.0.7` | 8 | 6
`/28` | `255.255.255.240` | `0.0.0.15` | 16 | 14
`/27` | `255.255.255.224` | `0.0.0.31` | 32 | 30
`/26` | `255.255.255.192` | `0.0.0.63` | 64 | 62
`/25` | `255.255.255.128` | `0.0.0.127` | 128 | 126
`/24` | `255.255.255.0` | `0.0.0.255` | 256 | 254
`/23` | `255.255.254.0` | `0.0.1.255` | 512 | 510
`/22` | `255.255.252.0` | `0.0.3.255` | 1024 | 1022
`/21` | `255.255.248.0` | `0.0.7.255` | 2048 | 2046
`/20` | `255.255.240.0` | `0.0.15.255` | 4096 | 4094
`/19` | `255.255.224.0` | `0.0.31.255` | 8192 | 8190
`/18` | `255.255.192.0` | `0.0.63.255` | 16384 | 16382
`/17` | `255.255.128.0` | `0.0.127.255` | 32768 | 32766
`/16` | `255.255.0.0` | `0.0.255.255` | 65536 | 65534
`/12` | `255.240.0.0` | `0.15.255.255` | 1.048.576 | 1.048.574
`/8`  | `255.0.0.0` | `0.255.255.255` | 16.777.216 | 16.777.214

## Ejemplo: `/27` con `192.168.0.64` (bloque de 32)
- Network: `192.168.0.64`
- Primera usable: `192.168.0.65`
- Última usable: `192.168.0.94`
- Broadcast: `192.168.0.95`

> Nota: el bloque siguiente sería `192.168.0.96/27` (usable .97–.126, broadcast .127), y después `192.168.0.128/27`.


## Reglas rápidas para convertir prefijo ↔ máscara

- Conversión prefijo → máscara (pasos mentales):
  1. Calcula cuántos octetos completos de `255` hay: `octetos_completos = floor(prefijo / 8)`.
  2. Calcula `r = prefijo % 8`. Si `r > 0`, el octeto parcial es la suma de los `r` bits más significativos: 128, 64, 32, ...
     - Ej.: `r = 3` → 128 + 64 + 32 = `224`.
  3. Los octetos restantes son `0`. Junta los octetos: `255`... + octeto_parcial + `0`...
  4. Resultado rápido: `/27` → `255.255.255.224`.

- Tabla rápida (bits en octeto parcial):
  - `r=0` → `0`
  - `r=1` → `128`
  - `r=2` → `192`
  - `r=3` → `224`
  - `r=4` → `240`
  - `r=5` → `248`
  - `r=6` → `252`
  - `r=7` → `254`
  - `r=8` → `255`

- Conversión máscara → prefijo (pasos mentales):
  1. Cuenta cuántos octetos `255` completos hay → multiplica por 8.
  2. Mira el primer octeto que no sea `255` y usa la tabla anterior para obtener cuántos bits aporta.
  3. Suma: `prefijo = 8*octetos_completos + bits_octeto_parcial`.
  - Ej.: `255.255.255.192` → 3 octetos completos = 24, `192` = 2 bits → `/26`.

- Trucos mnemotécnicos:
  - Memoriza **`/24 = 256`** direcciones. Subir 1 al prefijo → dividir por 2; bajar 1 → multiplicar por 2.
  - Secuencia para octeto parcial: `128, 192, 224, 240, 248, 252, 254, 255` (cada número añade la siguiente potencia de 2).
  - Tamaño de bloque = `256 - valor_octeto_parcial` (p. ej. `224` → bloque `32` → redes `.0, .32, .64, .96`).

- Ejemplos rápidos:
  - `/27` → `255.255.255.224` → bloque 32 → redes: `.0, .32, .64, .96`.
  - `255.255.255.240` → octeto parcial `240` → 4 bits → `/28`.

## Cómo calcular rápidamente a qué red pertenece una IP (reglas y ejemplos)

Regla rápida (pasos mentales):
- Identifica el octeto que cambia en la máscara (el primer octeto que no es `255`).
- Calcula el tamaño del bloque: `bloque = 256 - valor_del_octeto_parcial`.
- Calcula la red: `octeto_red = octeto_ip - (octeto_ip % bloque)`; los octetos inferiores quedan a `0`.
- Broadcast: `octeto_red + (bloque - 1)` en ese mismo octeto; octetos inferiores a `255`.

Mnemotecnia práctica:
- Si la máscara es `/24` o más pequeña (>= /24) la parte que cambia está en el último octeto: usa los bloques conocidos: `/25=128, /26=64, /27=32, /28=16, /29=8, /30=4, /31=2`.
- Si la máscara cambia en el tercer octeto (p. ej. `/22`, `/20`), aplica la misma idea pero al tercer octeto: por ejemplo `/22` → bloques de `4` en el tercer octeto (`.0, .4, .8, .12...`), `/20` → bloques de `16` (`.0, .16, .32...`).

Truco mental corto: "red = redondear hacia abajo al múltiplo del bloque" (resta el resto de la división por el bloque).

Ejemplos resueltos (varios, aleatorios):

1) IP: `63.17.112.244/26`
- Máscara `/26` → octeto parcial = `192` → bloque = `256 - 192 = 64`.
- Redes en último octeto: múltiplos de `64`: `0,64,128,192`.
- `244` está en el bloque `192` → Network = `63.17.112.192`, Broadcast = `63.17.112.255`.
- Usables: `63.17.112.193` … `63.17.112.254`.

2) IP: `10.5.23.77/27`
- `/27` → bloque = `32`. Último octeto múltiplos: `0,32,64,96`.
- `77` → pertenece al bloque `64`: Network = `10.5.23.64`, Broadcast = `10.5.23.95`.
- Usables: `10.5.23.65` … `10.5.23.94`.

3) IP: `172.16.245.10/28`
- `/28` → bloque = `16`. Último octeto múltiplos: `0,16,32,...`.
- `10` → pertenece al bloque `0`: Network = `172.16.245.0`, Broadcast = `172.16.245.15`.

4) IP: `192.0.2.130/25`
- `/25` → bloque = `128`. Último octeto: `0,128`.
- `130` → bloque `128`: Network = `192.0.2.128`, Broadcast = `192.0.2.255`.

5) IP: `203.0.113.17/30`
- `/30` → bloque = `4`. Último octeto: `...,12,16,20,...` (múltiplos de 4).
- `17` → bloque `16`: Network = `203.0.113.16`, Broadcast = `203.0.113.19`.
- Usables: `203.0.113.17` y `203.0.113.18`.

6) IP: `198.51.100.200/22`
- `/22` → máscara `255.255.252.0` → cambia el tercer octeto. Bloque en tercer octeto = `256 - 252 = 4`.
- Tercer octeto `100` → `100 % 4 = 0` → Network tercer octeto = `100`.
- Network = `198.51.100.0`, Broadcast = `198.51.103.255` (tercer octeto `100 + 3 = 103`, último octeto `255`).

7) IP: `10.10.1.15/20`
- `/20` → máscara `255.255.240.0` → bloque en tercer octeto = `16`.
- Tercer octeto `1` → `1 % 16 = 1` → red tercera = `1 - 1 = 0` → Network = `10.10.0.0`, Broadcast = `10.10.15.255`.

Notas especiales:
- `/31`: normalmente usado para enlaces punto a punto — no hay red/broadcast tradicionales; RFC3021 permite usar ambas direcciones.
- `/32`: dirección única (host).
