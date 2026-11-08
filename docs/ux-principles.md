# Hum — Principios de experiencia

> Hum debe poder usarla alguien que nunca abrió el Monitor de Actividad. Si una pantalla necesita explicación, la pantalla está mal.

Estos principios aplican a Mac y a Windows. Cada decisión de diseño y de código se revisa contra ellos.

---

## 1. Respuesta primero, datos después

Cada vista abre respondiendo una pregunta: **¿está todo bien?** Los números van debajo, para quien quiera profundizar.

- ✅ "Chrome está gastando la mitad de tu batería."
- ❌ Una tabla de 300 procesos ordenada por PID.

**Regla:** toda pantalla tiene una frase de estado arriba. Si no hay nada que decir, dice que todo está bien.

## 2. Cada problema llega con su acción

Nunca se muestra algo preocupante sin un botón para resolverlo justo al lado.

- ✅ "Spotify no responde" → **Forzar cierre**
- ❌ Un indicador rojo sin explicación ni salida.

**Regla:** si no hay acción posible, no se muestra como problema; a lo sumo como información.

## 3. Explicar antes de actuar

Todo proceso, disco o ítem responde tres preguntas en lenguaje humano:
1. **¿Qué es?** — "Ayuda a Spotlight a indexar tus archivos."
2. **¿Es seguro cerrarlo?** — Seguro / Revisa / Del sistema.
3. **¿Qué pasa si lo hago?** — "Se cerrará sin guardar los cambios abiertos."

**Regla:** ninguna acción destructiva sin que el usuario pueda ver antes qué va a pasar.

## 4. Acciones sin miedo

- **Cerrar** pide confirmación solo si hay riesgo real (documentos sin guardar). Si no, es inmediato.
- **Forzar cierre** siempre avisa que se puede perder trabajo.
- Lo que se borre (desde v3) va a la **Papelera**, nunca se elimina para siempre, y hay **Deshacer** visible.
- Los procesos del sistema muestran **candado + motivo**, no un error.

**Regla:** el usuario nunca debe sentir que puede romper su computador con Hum.

## 5. Permisos en contexto

- Se piden cuando se necesitan, no todos en el onboarding.
- Cada pedido dice **qué se gana** y **qué se ve sin el permiso**.
- Hum funciona sin permisos especiales; con ellos, ve más.
- Rechazar un permiso nunca bloquea la app ni se vuelve a pedir de forma insistente.

## 6. Lenguaje humano

- Sin jerga: "Gastando mucha batería", no "Energy Impact: 87.3".
- Sin culpa ni alarma: "Algo anda raro", no "¡ERROR CRÍTICO!".
- Cortas y concretas: un dato, una acción.
- Español e inglés escritos de forma nativa, no traducidos palabra por palabra.
- Voz de Hum: cercana y tranquila. *"All quiet. Your Mac is humming along."* (en Windows: *"Your PC is humming along."*)

**Regla:** los textos viven separados de la UI (archivo de contenido), para reutilizarlos en Windows y traducir sin tocar código.

## 7. Calma por defecto

- "Todo bien" no lleva color de estado. El color aparece solo cuando algo importa.
- Nada de animaciones, sonidos ni notificaciones que no aporten. Sonidos apagados por defecto.
- La onda se calma cuando se resuelve un problema: la recompensa es visual y silenciosa.

## 8. Simple para todos, profundo para quien lo busque

- Vista por defecto: **Apps** agrupadas, con nombre e ícono.
- Vista avanzada: **Todos los procesos**, a un clic, nunca impuesta.
- Los detalles técnicos (PID, ruta, uso exacto) aparecen al expandir, no antes.

## 9. Una sola identidad en Mac y Windows

- Hum se ve como Hum en cualquier sistema: mismos colores, onda, íconos, componentes y estilo visual, definidos en el Design System de Hum.
- Quien usa Hum en los dos sistemas reconoce la misma app de inmediato.
- Solo se adapta lo que el sistema impone o lo que el usuario espera por costumbre: dónde vive el ícono (barra de menú / bandeja del sistema), los controles de ventana, los atajos (⌘ / Ctrl) y los diálogos del sistema.
- Los componentes se definen por función (fila de proceso, tarjeta de métrica, acción), no por control de un sistema.

## 10. Para todas las personas

- Nada depende solo del color: cada métrica y estado lleva ícono y texto.
- Todo se puede hacer con teclado y con VoiceOver / Narrador.
- Contraste AA como mínimo en claro y oscuro.
- Funciona en equipos con y sin batería sin pantallas vacías ni ceros sin sentido.

## 11. Liviana de verdad

Hum no puede gastar lo que critica.
- En reposo, < ~1 % de CPU.
- Muestreo lento con la ventana cerrada, rápido solo cuando se mira.

---

## Checklist para revisar cada pantalla

- [ ] ¿Tiene una frase de estado arriba?
- [ ] ¿Cada problema tiene su acción al lado?
- [ ] ¿Se entiende qué es cada cosa sin saber de sistemas?
- [ ] ¿Las acciones destructivas avisan qué va a pasar?
- [ ] ¿Funciona sin permisos especiales?
- [ ] ¿Está diseñada en claro y oscuro, con y sin batería?
- [ ] ¿Hay estados de vacío, carga, error y sin permiso?
- [ ] ¿Se entiende sin depender del color?
- [ ] ¿Los textos están en ES y EN, separados de la UI?
- [ ] ¿Se vería y funcionaría igual en Windows?

## Cómo se valida

Cada fase de diseño termina con una prueba de usabilidad con 3 a 5 personas (idealmente que vienen de Windows). Una pantalla está lista cuando al menos 4 de 5 completan su tarea sin ayuda.
