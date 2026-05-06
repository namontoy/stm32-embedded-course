# Lista de verificación para el cambio de semana

**Antes de comenzar la semana N — haz esto cada vez, sin excepciones.**

---

## Pasos a seguir

**Paso 1 — Guarda y confirma tu trabajo actual.**

Antes de tocar cualquier rama, asegúrate de que todo lo que hiciste la semana pasada esté guardado de forma segura. Si haces esto primero, nada se perderá.

```bash
git add Src/ Inc/
git commit -m "semana-04: describe aquí lo que hiciste"
```

**Paso 2 — Cambia a la rama de la nueva semana.**

```bash
git checkout week-05
```

**Paso 3 — Trae tu trabajo anterior hacia adelante.**

```bash
git merge week-04
```

**Paso 4 — Resuelve el conflicto de configuración de la IA (siempre conserva la semana actual).**

Después del merge, los archivos de configuración pueden entrar en conflicto. Siempre resuelve a favor de la semana actual:

```bash
git checkout week-05 -- AGENTS.md ai-config/KNOWLEDGE.md
git commit -m "semana-05: código de semana-04 integrado, configuración IA de semana-05 conservada"
```

**Paso 5 — Verifica tu entorno.**

Abre STM32CubeIDE, confirma que el proyecto compila sin errores, y verifica que Copilot CLI reconoce el nuevo `AGENTS.md`. La IA ahora sabrá que estás en la semana 5.

**Paso 6 — Completa la autoevaluación inicial.**

Antes de escribir código nuevo, inicia una conversación con el asistente de IA. Te hará 3 o 4 preguntas de la semana anterior para confirmar que tu base es sólida. Si tienes dificultades con 2 o más preguntas, repasa el material antes de continuar.

---

## Solución de problemas

**"git me pide que confirme o guarde los cambios antes de cambiar de rama"**

Esto significa que tienes archivos modificados que aún no están confirmados. Git te está protegiendo — no te permitirá cambiar de rama y arriesgarte a perder trabajo. Regresa al Paso 1 y confirma todo primero. Si hiciste cambios que no quieres conservar, puedes descartarlos con `git checkout -- .`, pero ten cuidado, esto es irreversible.

**"Después del merge, el código de la semana anterior desapareció"**

Probablemente olvidaste el Paso 3, o lo ejecutaste en el orden incorrecto. Verifica en qué rama estás con `git branch` y revisa el historial de confirmaciones con `git log --oneline`. Tu trabajo casi con certeza no se perdió — todavía está en la rama de la semana anterior. Regresa, verifica que esté allí y repite la lista desde el Paso 2.

**"git dice que hay conflictos de merge en Src/ o Inc/"**

Esto es poco frecuente pero puede ocurrir si el mismo archivo fue modificado en ambas ramas. Abre el archivo en conflicto — git habrá marcado las secciones conflictivas con `<<<<<<<`, `=======` y `>>>>>>>`. Lee ambas versiones con cuidado y conserva la que representa tu código más reciente. Luego marca el conflicto como resuelto:

```bash
git add Src/el_archivo_en_conflicto.c
git commit -m "semana-05: conflicto de merge resuelto en archivo"
```

Si no estás seguro de cuál versión conservar, pregúntale a tu asistente de IA — esta es exactamente el tipo de situación donde hablar en voz alta sobre el problema ayuda.

**"El asistente de IA sigue comportándose como la semana pasada"**

Esto significa que Copilot CLI no está reconociendo el nuevo `AGENTS.md`. Verifica que completaste el Paso 4 correctamente revisando el contenido del archivo: `cat AGENTS.md`. Si el archivo todavía muestra el contexto de conocimiento de la semana pasada, repite el Paso 4. Si el archivo se ve correcto pero la IA sigue comportándose de forma extraña, cierra y vuelve a abrir la sesión de terminal.

**"git merge dice 'Already up to date'"**

Esto en realidad significa que todo está bien — git verificó y encontró que `week-05` ya contiene todo lo de `week-04`. Esto puede ocurrir si ya habías hecho el merge anteriormente. Puedes continuar tranquilamente al Paso 5.

**"Confirmé algo en la rama equivocada"**

No entres en pánico — este es uno de los errores más comunes en git y es completamente recuperable. Llama a tu profesor antes de intentar corregirlo por tu cuenta, porque el comando de recuperación (`git cherry-pick` o `git reset`) depende exactamente de lo que ocurrió. Intentar corregirlo sin entender la situación puede hacer las cosas más difíciles de recuperar.
