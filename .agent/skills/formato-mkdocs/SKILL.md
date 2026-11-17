---
name: formato-mkdocs
description: Redacta, estructura y formatea apuntes técnicos siguiendo las convenciones de Material for MkDocs. Úsala siempre que el usuario pida crear o editar temas, unidades didácticas, ejercicios o documentación en Markdown dentro de docs/.
---

# Estándares de redacción para Material for MkDocs

Cuando crees o modifiques archivos dentro de `docs/`, cumple estrictamente estas reglas:

1. **Sintaxis de Admonitions (PyMdown Extensions):**
   - Usa `!!! note`, `!!! warning` o `!!! tip` con indentación de 4 espacios para destacar conceptos clave, advertencias o buenas prácticas.
   - Para ejercicios con solución oculta, usa bloques desplegables `??? example "Solución"`.

2. **Bloques de código y pestañas:**
   - Especifica siempre el lenguaje tras los tres acentos graves (` ```bash `, ` ```yaml `, ` ```python `) y añade el título cuando sea un archivo de configuración (` ```ini title="/etc/ssh/sshd_config" `).
   - Agrupa alternativas (ej. Debian vs. Windows) con pestañas de contenido (`=== "Linux"` / `=== "Windows"`).

3. **Listas y anidamiento (Python-Markdown):**
   - Deja siempre **una línea en blanco** antes de iniciar cualquier lista numerada o con viñetas (especialmente tras un párrafo o encabezado) para que el parser la renderice correctamente.
   - Si incluyes listas o bloques de código dentro de un *admonition* (`!!!` / `???`) o una pestaña (`===`), mantén tanto la línea en blanco previa como la indentación de 4 espacios.

4. **Fórmulas matemáticas:**
   - No utilices sintaxis LaTeX (`$` o `$$`). Representa fórmulas, operadores y expresiones matemáticas usando Markdown estándar (subíndices/superíndices HTML `<sub>`/`<sup>`, caracteres Unicode o bloques de código).

5. **Enlaces y contenido multimedia:**
   - En **enlaces externos** (HTTP/HTTPS), añade siempre `{target="_blank"}` al final (ej. `[Texto](https://ejemplo.com){target="_blank"}`, requiere extensión `attr_list`). No lo añadas en enlaces internos relativos entre archivos `.md`.
   - Incrusta vídeos de YouTube únicamente con el formato del plugin `mkdocs-video`: `![type:video](https://www.youtube.com/embed/CODIGOVIDEO)`.

6. **Navegación y validación:**
   - Antes de dar por terminada cualquier tarea estructural, ejecuta `mkdocs build --strict` en la terminal integrada para verificar que no existen enlaces rotos ni advertencias de sintaxis.

7. **Esquemas y diagramas (Mermaid):**
   - Cuando el contenido requiera diagramas de flujo, arquitecturas de red, secuencias o esquemas conceptuales, represéntalos usando bloques de código con el lenguaje `mermaid` (` ```mermaid `).
   - Entrecomilla siempre las etiquetas de los nodos que contengan paréntesis, barras o caracteres especiales (ej. `A["Servidor (Ubuntu)"]` en lugar de `A[Servidor (Ubuntu)]`) para evitar errores de renderizado.
   - No fuerces colores hexadecimales fijos en los nodos para respetar el cambio automático entre modo claro y oscuro de Material for MkDocs.

8. **Control de activación:**
   - Al finalizar tu respuesta en el chat, añade exactamente la línea: `[Skill aplicada: formato-mkdocs]`.