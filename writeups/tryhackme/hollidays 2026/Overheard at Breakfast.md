# TryHackMe - Overheard at Breakfast

**Dificultad:** Fácil
**Categoría:** OSINT
**Objetivo:** Localizar una cuenta oculta a partir de una conversación aparentemente inocente.

---

# Introducción

En este reto escuchamos una conversación entre dos personas durante un desayuno. A simple vista parece una charla cualquiera, pero contiene suficientes pistas como para localizar un perfil que su propietario nunca pretendía hacer público.

Este tipo de retos enseña una lección muy importante en OSINT:

> Una sola dirección de correo puede convertirse en la llave para descubrir toda la identidad digital de una persona.

---

# Análisis de la conversación

Durante la conversación aparecen varios detalles interesantes.

Uno de los personajes comenta que utiliza un servicio cuyo nombre comienza por la letra **"G"** para gestionar su perfil o iniciar sesión en distintas plataformas.

Además, menciona directamente su dirección de correo:

```text
lambobytelotushotel@gmail.com
```

Ese será nuestro punto de partida.

---

# Enumeración del correo electrónico

La primera herramienta utilizada fue **Epieos**, especializada en recopilar información pública asociada a direcciones de correo.

```
https://epieos.com/
```

La búsqueda revela varios servicios vinculados a la cuenta.

Entre ellos aparecen referencias a:

* Google
* Otros servicios donde utiliza ese correo
* Una cuenta de **Gravatar**

---

# ¿Qué es Gravatar?

Gravatar (**Globally Recognized Avatar**) es un servicio que asocia una imagen de perfil a una dirección de correo electrónico.

Muchísimas plataformas (como WordPress) muestran automáticamente ese avatar cuando el usuario comenta o publica contenido.

Gracias a ello podemos acceder directamente al perfil público asociado.

---

# Accediendo al perfil

El perfil encontrado fue:

```text
https://gravatar.com/d43faafe9d7f056793bd037b8d6e321acad985c222d83775b10d6539e301e931
```

Dentro del perfil aparece una cadena bastante sospechosa:

```text
VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9
```

Tiene toda la pinta de estar codificada en Base64.

---

# Decodificando la cadena

Se puede utilizar cualquier decodificador Base64.

Por ejemplo:

```
https://www.base64decode.org/
```

Al decodificar el contenido obtenemos directamente la flag.

---

# Flag

```text
THM{S3creT_Pr0fil3_H4s_b33n_Ident1fi3d}
```

---

# Investigación adicional

Durante la investigación también apareció una referencia interesante relacionada con un antiguo perfil de Google Plus:

```text
https://web.archive.org/web/*/plus.google.com/109541557676124188877*
```

Aunque finalmente no fue necesaria para resolver el reto, queda pendiente investigar si la Wayback Machine conserva información útil sobre ese perfil y cómo podría utilizarse durante una investigación OSINT real.

---

# Herramientas utilizadas

* Epieos
* Gravatar
* Base64 Decoder
* Wayback Machine (investigación adicional)

---

# Conceptos aprendidos

* Pivotar desde una dirección de correo electrónico.
* Descubrir servicios asociados a una identidad digital.
* Uso de Gravatar durante investigaciones OSINT.
* Identificación de cadenas codificadas en Base64.
* La importancia de correlacionar pequeñas pistas aparentemente insignificantes.

---

# Reflexión personal

Este reto fue bastante rápido de completar, pero deja una enseñanza muy útil: una única dirección de correo puede revelar mucha más información de la que la mayoría de la gente imagina.

No hizo falta realizar ataques ni técnicas complejas; simplemente seguir el rastro de la información pública disponible y enlazar cada pista con la siguiente.

Es un buen ejemplo de cómo funciona una investigación OSINT real: pequeñas piezas de información que, unidas, terminan revelando la identidad digital de una persona.
