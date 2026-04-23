# Cómo exportar los diagramas Mermaid a JPG

Los diagramas están en formato **Mermaid** dentro de bloques de código en los archivos `.md`. Hay **3 formas** de convertirlos a JPG según tu preferencia.

---

## Método 1 (RECOMENDADO) · Mermaid Live Editor online

**Ventajas:** Gratis, no instala nada, lo tendrás listo en 2 minutos.

### Pasos

1. Abre https://mermaid.live en tu navegador.
2. Abre el archivo `.md` del diagrama que quieres exportar (por ejemplo, `diagramas/01-casos-de-uso.md`).
3. Copia el contenido que está **entre** las líneas ` ```mermaid ` y ` ``` ` (solo el código del diagrama).
4. Pega ese contenido en el panel izquierdo de Mermaid Live.
5. El diagrama se renderiza automáticamente a la derecha.
6. Click en el botón **"Actions"** arriba a la derecha.
7. Selecciona **"PNG"** (o **"SVG"** si quieres calidad vectorial).
8. Para convertir PNG a JPG:
   - Windows: Abre el PNG con "Paint" → Archivo → Guardar como → "JPEG".
   - O usa https://convertio.co/png-jpg/ (gratis, online).

### Repite por cada diagrama

Repite los pasos 2–8 para los 9 archivos de diagrama:

- `01-casos-de-uso.md`
- `02-clases.md` (2 diagramas adentro)
- `03-c4-contexto.md`
- `04-c4-contenedores.md`
- `05-c4-componentes.md`
- `06-base-de-datos.md`
- `07-arquitectura-backend.md` (3 diagramas adentro)
- `08-arquitectura-frontend.md` (3 diagramas adentro)
- `09-flujo-pedido-secuencia.md`

Resultado: ~13 archivos JPG guardados en la carpeta `documentacion/diagramas/exportados/`.

---

## Método 2 · Mermaid CLI (automático)

**Ventajas:** Convierte todos los diagramas con un solo comando. Ideal si vas a regenerar los diagramas múltiples veces.

### Requisito: Node.js instalado

### Instalación

```bash
npm install -g @mermaid-js/mermaid-cli
```

### Convertir un archivo .mmd a JPG

Primero hay que **extraer** el bloque Mermaid del `.md` a un archivo `.mmd`. Ejemplo:

```bash
# Crear directorio de salida
mkdir -p documentacion/diagramas/exportados

# Extraer el diagrama (requiere copiar manualmente el contenido)
# Luego convertir:
mmdc -i diagrama.mmd -o diagrama.jpg -w 1600 -H 1200
```

### Script automatizado (PowerShell Windows)

Guarda como `convert-diagrams.ps1` en la carpeta `documentacion/diagramas/`:

```powershell
# Requisitos: @mermaid-js/mermaid-cli instalado globalmente
$outputDir = ".\exportados"
New-Item -ItemType Directory -Force -Path $outputDir | Out-Null

Get-ChildItem -Filter "*.md" | ForEach-Object {
    $mdFile = $_
    $content = Get-Content $mdFile.FullName -Raw

    # Extraer bloques de código ```mermaid ... ```
    $pattern = '(?s)```mermaid\s*(.*?)\s*```'
    $matches = [regex]::Matches($content, $pattern)

    $idx = 0
    foreach ($m in $matches) {
        $idx++
        $mmdContent = $m.Groups[1].Value.Trim()
        $baseName = [System.IO.Path]::GetFileNameWithoutExtension($mdFile.Name)
        $tempMmd = "$outputDir\$baseName-$idx.mmd"
        $outputJpg = "$outputDir\$baseName-$idx.jpg"

        Set-Content -Path $tempMmd -Value $mmdContent
        mmdc -i $tempMmd -o $outputJpg -w 1600 -H 1200
        Remove-Item $tempMmd
        Write-Host "Generado: $outputJpg"
    }
}
```

Ejecutar:

```powershell
cd documentacion/diagramas
./convert-diagrams.ps1
```

---

## Método 3 · VS Code con extensión

Si usas VS Code, instala la extensión:

- **Markdown Preview Mermaid Support** (id: `bierner.markdown-mermaid`)
- O **Mermaid Markdown Syntax Highlighting**

Luego:

1. Abre cualquier `.md` con diagramas Mermaid.
2. Pulsa `Ctrl+Shift+V` para abrir el preview.
3. Click derecho sobre el diagrama renderizado → "Guardar imagen como..."
4. Elige formato JPG.

---

## Comparación de métodos

| Método | Tiempo | Facilidad | Calidad | Requiere instalar |
|---|---|---|---|---|
| 1 · Mermaid Live | 2 min/diagrama | ⭐⭐⭐⭐⭐ | Excelente | No |
| 2 · CLI + script | 10 min setup, <1 min ejecución | ⭐⭐⭐ | Excelente | Sí (Node.js + mmdc) |
| 3 · VS Code | 1 min/diagrama | ⭐⭐⭐⭐ | Buena | Sí (extensión) |

## Recomendación final

Para el entregable académico, **usa el Método 1 (Mermaid Live)**. Son 13 imágenes aproximadamente, termináis en ~15-20 minutos totales y obtenéis calidad profesional. Guárdalos en una carpeta `exportados/` con nombres descriptivos:

```
documentacion/diagramas/exportados/
├── 01-casos-de-uso.jpg
├── 02-clases-backend.jpg
├── 02-clases-frontend.jpg
├── 03-c4-contexto.jpg
├── 04-c4-contenedores.jpg
├── 05-c4-componentes.jpg
├── 06-base-de-datos.jpg
├── 07-arquitectura-backend-capas.jpg
├── 07-arquitectura-backend-modulos.jpg
├── 07-arquitectura-backend-secuencia.jpg
├── 08-arquitectura-frontend-capas.jpg
├── 08-arquitectura-frontend-home.jpg
├── 08-arquitectura-frontend-admin.jpg
└── 09-flujo-pedido-secuencia.jpg
```
