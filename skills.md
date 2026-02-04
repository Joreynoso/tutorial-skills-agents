# Tutorial: Cómo Crear un Skill para MCP
## Ejemplo: Generación de Imágenes con DALL-E

Este tutorial te guiará paso a paso en la creación de un **skill** profesional para un agente que soporte el sistema de Skills. Usaremos como ejemplo un skill de generación de imágenes con DALL-E.

---

## 📋 Tabla de Contenidos

1. [¿Qué es un Skill?](#qué-es-un-skill)
2. [Estructura de un Skill](#estructura-de-un-skill)
3. [Paso 1: Planificación](#paso-1-planificación)
4. [Paso 2: Crear la Estructura](#paso-2-crear-la-estructura)
5. [Paso 3: Escribir el SKILL.md](#paso-3-escribir-el-skillmd)
6. [Paso 4: Agregar Scripts](#paso-4-agregar-scripts)
7. [Paso 5: Referencias y Assets](#paso-5-referencias-y-assets)
8. [Paso 6: Validación y Empaquetado](#paso-6-validación-y-empaquetado)
9. [Mejores Prácticas](#mejores-prácticas)
10. [Antipatrones (Qué Evitar)](#antipatrones-qué-evitar)

---

## ¿Qué es un Skill?

Un **skill** es un paquete modular que extiende las capacidades de un agente de IA proporcionando:

- **Conocimiento especializado** - Información específica del dominio
- **Flujos de trabajo** - Procedimientos paso a paso
- **Integraciones con herramientas** - Scripts, APIs, y formatos de archivo
- **Recursos bundleados** - Templates, referencias, y assets

Piensa en un skill como una "guía de incorporación" que transforma a Claude de un agente genérico en un especialista.

---

## Estructura de un Skill

Todo skill tiene esta estructura básica:

```
image-generator/
├── SKILL.md              # (OBLIGATORIO) Archivo principal con instrucciones
├── scripts/              # (OPCIONAL) Código ejecutable
│   └── generate_image.py
├── references/           # (OPCIONAL) Documentación de referencia
│   └── dalle_api_docs.md
└── assets/              # (OPCIONAL) Archivos usados en output
    └── default_prompts.txt
```

### Componentes Clave

#### 1. **SKILL.md** (OBLIGATORIO)
- Contiene el frontmatter YAML con metadata
- Instrucciones en Markdown para usar el skill
- Única parte siempre cargada en contexto

#### 2. **scripts/** (OPCIONAL)
- Código Python, Bash, etc.
- Para tareas que requieren ejecución determinista
- Se pueden ejecutar sin cargar en contexto

#### 3. **references/** (OPCIONAL)
- Documentación detallada
- Se carga solo cuando Claude lo necesita
- Mantiene el SKILL.md ligero

#### 4. **assets/** (OPCIONAL)
- Templates, imágenes, fonts, etc.
- NO se cargan en contexto
- Se usan directamente en el output

---

## Paso 1: Planificación

Antes de crear el skill, responde estas preguntas:

### 1.1 Definir Casos de Uso

**Para nuestro ejemplo de image-generator:**

```
¿Qué hará el skill?
→ Generar imágenes usando DALL-E 3 API

¿Qué consultas de usuario activarán este skill?
→ "Genera una imagen de..."
→ "Crea un logo para..."
→ "Hazme una ilustración de..."
→ "Diseña una imagen..."

¿Qué funcionalidad necesita?
→ Llamadas a API de DALL-E
→ Manejo de prompts
→ Gestión de API keys
→ Descarga y guardado de imágenes
→ Manejo de errores
```

### 1.2 Identificar Recursos Reusables

Analiza qué recursos necesitas incluir:

| Recurso | Tipo | Justificación |
|---------|------|---------------|
| Script de generación | `scripts/` | El código de API se reescribiría cada vez |
| Documentación API | `references/` | Los parámetros pueden olvidarse |
| Prompts de ejemplo | `assets/` | Ayuda a usuarios con ideas |

---

## Paso 2: Crear la Estructura

### 2.1 Crear el Directorio Principal

```bash
mkdir image-generator
cd image-generator
```

### 2.2 Crear Subdirectorios

```bash
mkdir scripts
mkdir references
mkdir assets
```

### 2.3 Convenciones de Nombres

**✅ CORRECTO:**
- `image-generator` (kebab-case)
- `pdf-editor`
- `api-helper`

**❌ INCORRECTO:**
- `ImageGenerator` (PascalCase)
- `image_generator` (snake_case)
- `Image Generator` (espacios)

**Reglas:**
- Solo minúsculas
- Guiones como separadores
- Sin espacios ni caracteres especiales
- Máximo 64 caracteres

---

## Paso 3: Escribir el SKILL.md

El archivo `SKILL.md` tiene dos partes: **frontmatter YAML** y **cuerpo Markdown**.

### 3.1 Frontmatter YAML (Metadata)

El frontmatter va al inicio del archivo entre `---`:

```yaml
---
name: image-generator
description: Generate AI images using DALL-E 3 API. Use this skill when users request image generation, illustration creation, logo design, or visual content creation. Triggers include phrases like "generate an image", "create a picture", "design a logo", "make an illustration", or any request for visual content creation. Supports customization of style, size, and quality parameters.
---
```

#### Campos Obligatorios:

**`name:`**
- Mismo nombre que el directorio
- Kebab-case
- Identificador único

**`description:`**
- **MUY IMPORTANTE:** Es el mecanismo principal de activación
- Debe incluir:
  - ✅ QUÉ hace el skill
  - ✅ CUÁNDO usarlo (triggers específicos)
  - ✅ Contextos de uso
  - ✅ Tipos de archivo o tareas
- Claude SOLO lee esto para decidir si usar el skill
- No incluyas información de "cuándo usar" en el cuerpo

#### Campos Opcionales:

```yaml
license: MIT
compatibility: Requires OpenAI API key in environment
```

### 3.2 Cuerpo del SKILL.md

**Ejemplo completo para nuestro skill:**

```markdown
# Image Generator

## Overview

This skill enables AI image generation using OpenAI's DALL-E 3 API. It handles prompt engineering, API communication, image downloading, and file management.

## Quick Start

### Basic Usage

Generate an image with default settings:

```python
from scripts.generate_image import generate_image

# Simple generation
image_path = generate_image(
    prompt="A serene mountain landscape at sunset",
    api_key="your-api-key"
)
```

### With Custom Parameters

```python
image_path = generate_image(
    prompt="A futuristic city with flying cars",
    api_key="your-api-key",
    size="1792x1024",  # "1024x1024", "1792x1024", "1024x1792"
    quality="hd",       # "standard" or "hd"
    style="vivid"       # "vivid" or "natural"
)
```

## Workflow

### Step 1: Understand User Intent

Analyze the user's request to determine:
- Subject matter
- Desired style (photorealistic, artistic, cartoon, etc.)
- Image dimensions
- Quality requirements

### Step 2: Craft the Prompt

**Best practices:**
- Be specific and descriptive
- Include style, mood, and composition details
- Mention lighting, colors, and atmosphere
- For logos: specify simplicity, colors, and format needs

**Example transformations:**

| User Request | Optimized Prompt |
|--------------|------------------|
| "Make a logo for my bakery" | "A minimalist logo for an artisan bakery, featuring a wheat stalk and rolling pin, warm earthy colors, clean lines, suitable for business cards and signage" |
| "Picture of a dog" | "A golden retriever puppy sitting in a sunny meadow, soft natural lighting, photorealistic style, shallow depth of field, warm and inviting atmosphere" |

### Step 3: Call the API

Use the provided script with error handling:

```python
try:
    image_path = generate_image(
        prompt=enhanced_prompt,
        api_key=os.getenv("OPENAI_API_KEY"),
        size="1024x1024"
    )
    print(f"Image saved to: {image_path}")
except Exception as e:
    print(f"Generation failed: {e}")
```

### Step 4: Deliver Results

- Save image to `/mnt/user-data/outputs/`
- Provide download link to user
- Optionally show thumbnail if supported
- Share prompt used for reference

## API Parameters

### Size Options

- `1024x1024` - Square (default)
- `1792x1024` - Landscape
- `1024x1792` - Portrait

### Quality Options

- `standard` - Faster, lower cost
- `hd` - Higher detail, slower

### Style Options

- `vivid` - More dramatic and artistic
- `natural` - More realistic and subdued

## Error Handling

Common errors and solutions:

**API Key Issues:**
```
Error: Invalid API key
Solution: Verify OPENAI_API_KEY environment variable
```

**Content Policy Violations:**
```
Error: Your request was rejected as a result of our safety system
Solution: Modify prompt to remove potentially unsafe content
```

**Rate Limits:**
```
Error: Rate limit exceeded
Solution: Wait 60 seconds and retry
```

## Security Best Practices

1. **Never hardcode API keys** - Use environment variables
2. **Validate user prompts** - Check for injection attempts
3. **Set usage limits** - Monitor API costs
4. **Store keys securely** - Use secret management systems

## Resources

### Scripts
- `scripts/generate_image.py` - Main generation script with full API integration

### References
- `references/dalle_api_docs.md` - Complete DALL-E 3 API documentation
- `references/prompt_engineering.md` - Advanced prompt crafting techniques

### Assets
- `assets/example_prompts.txt` - Curated collection of effective prompts
```

### 3.3 Principios de Escritura

**✅ Usa forma imperativa/infinitiva:**
```markdown
✓ Generate an image using the API
✓ Validate the prompt before sending
✓ Handle errors gracefully

✗ You should generate an image
✗ The user needs to validate
✗ It is recommended to handle errors
```

**✅ Sé conciso:**
- El context window es limitado
- Asume que Claude es inteligente
- Solo añade información no-obvia
- Prefiere ejemplos sobre explicaciones largas

**✅ Divide contenido largo:**
- Si SKILL.md > 500 líneas, usa `references/`
- Mantén workflow en SKILL.md
- Mueve detalles técnicos a referencias

---

## Paso 4: Agregar Scripts

### 4.1 Crear el Script Principal

**`scripts/generate_image.py`:**

```python
#!/usr/bin/env python3
"""
DALL-E 3 Image Generator
Handles API communication, image generation, and file management.
"""

import os
import requests
from pathlib import Path
from datetime import datetime


def generate_image(
    prompt: str,
    api_key: str,
    size: str = "1024x1024",
    quality: str = "standard",
    style: str = "vivid",
    output_dir: str = "/mnt/user-data/outputs"
) -> str:
    """
    Generate an image using DALL-E 3 API.
    
    Args:
        prompt: Text description of desired image
        api_key: OpenAI API key
        size: Image dimensions ("1024x1024", "1792x1024", "1024x1792")
        quality: Image quality ("standard" or "hd")
        style: Image style ("vivid" or "natural")
        output_dir: Directory to save generated image
        
    Returns:
        Path to saved image file
        
    Raises:
        ValueError: Invalid parameters
        requests.HTTPError: API request failed
    """
    # Validate parameters
    valid_sizes = ["1024x1024", "1792x1024", "1024x1792"]
    if size not in valid_sizes:
        raise ValueError(f"Invalid size. Must be one of: {valid_sizes}")
        
    valid_qualities = ["standard", "hd"]
    if quality not in valid_qualities:
        raise ValueError(f"Invalid quality. Must be one of: {valid_qualities}")
        
    valid_styles = ["vivid", "natural"]
    if style not in valid_styles:
        raise ValueError(f"Invalid style. Must be one of: {valid_styles}")
    
    # Prepare API request
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "model": "dall-e-3",
        "prompt": prompt,
        "size": size,
        "quality": quality,
        "style": style,
        "n": 1
    }
    
    # Call API
    print(f"Generating image with DALL-E 3...")
    print(f"Prompt: {prompt[:100]}...")
    
    response = requests.post(
        "https://api.openai.com/v1/images/generations",
        headers=headers,
        json=payload,
        timeout=60
    )
    
    response.raise_for_status()
    result = response.json()
    
    # Get image URL
    image_url = result["data"][0]["url"]
    revised_prompt = result["data"][0].get("revised_prompt", prompt)
    
    # Download image
    print(f"Downloading image...")
    image_response = requests.get(image_url, timeout=30)
    image_response.raise_for_status()
    
    # Save to file
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"dalle_image_{timestamp}.png"
    output_path = Path(output_dir) / filename
    
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_bytes(image_response.content)
    
    print(f"✓ Image saved to: {output_path}")
    print(f"✓ Revised prompt: {revised_prompt}")
    
    return str(output_path)


def main():
    """Command-line interface for testing."""
    import sys
    
    if len(sys.argv) < 2:
        print("Usage: generate_image.py <prompt> [size] [quality] [style]")
        print("Example: generate_image.py 'A serene lake' 1024x1024 hd vivid")
        sys.exit(1)
    
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        print("Error: OPENAI_API_KEY environment variable not set")
        sys.exit(1)
    
    prompt = sys.argv[1]
    size = sys.argv[2] if len(sys.argv) > 2 else "1024x1024"
    quality = sys.argv[3] if len(sys.argv) > 3 else "standard"
    style = sys.argv[4] if len(sys.argv) > 4 else "vivid"
    
    try:
        image_path = generate_image(prompt, api_key, size, quality, style)
        print(f"\n✓ Success! Image: {image_path}")
    except Exception as e:
        print(f"\n✗ Error: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

### 4.2 Hacer el Script Ejecutable

```bash
chmod +x scripts/generate_image.py
```

### 4.3 Probar el Script

```bash
# Set API key
export OPENAI_API_KEY="sk-..."

# Test generation
python scripts/generate_image.py "A beautiful sunset over mountains"
```

**✅ IMPORTANTE:** Siempre testea los scripts antes de incluirlos en el skill.

---

## Paso 5: Referencias y Assets

### 5.1 Referencias (Documentación Detallada)

**`references/dalle_api_docs.md`:**

```markdown
# DALL-E 3 API Reference

## Endpoint

```
POST https://api.openai.com/v1/images/generations
```

## Authentication

Include API key in header:
```
Authorization: Bearer YOUR_API_KEY
```

## Request Body

| Parameter | Type | Required | Options | Description |
|-----------|------|----------|---------|-------------|
| model | string | Yes | "dall-e-3" | Model identifier |
| prompt | string | Yes | - | Text description (max 4000 chars) |
| size | string | No | "1024x1024", "1792x1024", "1024x1792" | Output dimensions |
| quality | string | No | "standard", "hd" | Image quality |
| style | string | No | "vivid", "natural" | Visual style |
| n | integer | No | 1 | Number of images (DALL-E 3 only supports 1) |

## Response Format

```json
{
  "created": 1699551993,
  "data": [
    {
      "url": "https://oaidalleapiprodscus...",
      "revised_prompt": "A more detailed version..."
    }
  ]
}
```

## Rate Limits

- Free tier: 3 images/minute
- Tier 1: 5 images/minute
- Tier 2: 7 images/minute

## Error Codes

- 400: Invalid request
- 401: Invalid API key
- 429: Rate limit exceeded
- 500: Server error
```

**`references/prompt_engineering.md`:**

```markdown
# Prompt Engineering for DALL-E 3

## Core Principles

1. **Be Specific** - More details = better results
2. **Include Style Descriptors** - "photorealistic", "oil painting", "minimalist"
3. **Mention Composition** - "centered", "aerial view", "close-up"
4. **Describe Lighting** - "golden hour", "dramatic shadows", "soft diffused light"
5. **Add Mood/Atmosphere** - "serene", "energetic", "mysterious"

## Examples

### Photography Style
```
"A candid portrait of a chef in a bustling kitchen, 
shallow depth of field, warm ambient lighting, 
documentary photography style, 50mm lens perspective"
```

### Illustration Style
```
"An isometric illustration of a smart city, 
clean vector art style, bright pastel colors, 
white background, featuring autonomous vehicles 
and green spaces, modern and optimistic mood"
```

### Logo Design
```
"A minimalist logo for a tech startup, 
incorporating a circuit board pattern and leaf motif, 
geometric shapes, teal and gray color palette, 
simple and memorable, suitable for app icon"
```

## Common Mistakes

❌ Too vague: "A nice picture of a dog"
✓ Specific: "A golden retriever puppy playing in autumn leaves, natural lighting, joyful expression, photorealistic"

❌ Conflicting styles: "A photorealistic cartoon"
✓ Consistent: "A Pixar-style 3D animated character"

❌ Overloaded: 200-word prompt with every detail
✓ Focused: 2-3 key descriptors per aspect (subject, style, composition, mood)
```

### 5.2 Assets (Recursos de Output)

**`assets/example_prompts.txt`:**

```
# Example Prompts for Common Use Cases

## Marketing & Branding
"A modern minimalist logo for a coffee shop, incorporating a coffee bean and steam, warm brown tones, simple geometric shapes"

"A hero image for a SaaS website, featuring abstract data visualizations and connected nodes, gradient blue background, professional and tech-forward"

## Social Media
"An Instagram-style photo of a latte with foam art, marble table surface, natural window lighting, shallow depth of field, cozy cafe aesthetic"

"A motivational quote image with the text 'Dream Big' in elegant typography, mountain landscape background, cinematic lighting, inspirational mood"

## Product Visualization
"A product shot of wireless earbuds on a minimal white background, dramatic studio lighting with soft shadows, commercial photography style"

"An exploded view technical illustration of a smartwatch, clean line art style, labeled components, engineering drawing aesthetic"

## Creative Art
"A surreal landscape painting combining elements of desert and ocean, Salvador Dali style, dreamlike atmosphere, vibrant sunset colors"

"A cyberpunk cityscape at night, neon signs reflecting on wet streets, flying cars, detailed architecture, cinematic composition"
```

---

## Paso 6: Validación y Empaquetadoo

### 6.1 Estructura Final del Proyecto

Verifica que tu estructura se vea así:

```
image-generator/
├── SKILL.md
├── scripts/
│   └── generate_image.py
├── references/
│   ├── dalle_api_docs.md
│   └── prompt_engineering.md
└── assets/
    └── example_prompts.txt
```

### 6.2 Checklist de Validación

**SKILL.md:**
- [ ] Frontmatter YAML correcto
- [ ] `name:` coincide con nombre de directorio
- [ ] `description:` es completo e incluye triggers
- [ ] Cuerpo tiene Overview claro
- [ ] Workflow está documentado
- [ ] Ejemplos de código incluidos
- [ ] Menos de 500 líneas

**Scripts:**
- [ ] Son ejecutables (chmod +x)
- [ ] Están testeados y funcionan
- [ ] Tienen docstrings
- [ ] Manejan errores apropiadamente

**Referencias:**
- [ ] Contienen info que complementa SKILL.md
- [ ] No duplican contenido del SKILL.md
- [ ] Están referenciadas desde SKILL.md

**Assets:**
- [ ] Son archivos que se usan en output
- [ ] No son documentación

### 6.3 Empaquetar el Skill

Si tienes acceso al script `package_skill.py`:

```bash
python /path/to/package_skill.py image-generator
```

Esto creará `image-generator.skill` (un archivo ZIP con extensión .skill).

**Manualmente:**

```bash
cd image-generator
zip -r image-generator.skill .
```

---

## Mejores Prácticas

### 1. Progressive Disclosure (Revelación Progresiva)

El sistema carga información en niveles:

1. **Metadata** (siempre): `name` + `description` (~100 palabras)
2. **SKILL.md body** (cuando se activa): <5000 palabras
3. **References** (cuando Claude lo necesita): Ilimitado

**Aplica esto:**
- Mantén SKILL.md conciso
- Mueve detalles a `references/`
- Claude cargará referencias solo si las necesita

### 2. Write for Another Claude

Tu audiencia es otra instancia de Claude, no humanos:

```markdown
❌ "You might want to consider checking the API key"
✓ "Validate API key before making requests"

❌ "It's generally a good idea to handle errors"
✓ "Wrap API calls in try-except blocks"
```

### 3. Degrees of Freedom

Ajusta cuánta libertad le das a Claude:

**Alta libertad** (instrucciones texto):
```markdown
Analyze the user's request and determine appropriate image parameters.
Consider style, composition, and mood.
```

**Libertad media** (pseudocódigo):
```python
if user_mentions_logo:
    use minimal style and simple composition
elif user_mentions_photo:
    use photorealistic style with natural lighting
```

**Baja libertad** (script específico):
```python
# Use this exact script - do not modify
result = generate_image(prompt, api_key, "1024x1024", "hd", "vivid")
```

### 4. Token Efficiency

Cada palabra cuenta:

❌ **Verboso** (50 tokens):
```markdown
In order to generate an image, you should first validate that the
API key is present in the environment. After that, you need to 
construct the prompt carefully. Then you should call the API with
the appropriate parameters.
```

✓ **Conciso** (20 tokens):
```markdown
1. Validate API key exists
2. Construct prompt
3. Call API with parameters
```

### 5. Concrete Examples Over Explanations

❌ **Explicación abstracta**:
```markdown
Make sure to include details about the subject, style, composition,
lighting, and mood in your prompts for best results.
```

✓ **Ejemplo concreto**:
```markdown
Example transformation:
"a dog" → "A golden retriever puppy in a sunny meadow, 
photorealistic, natural lighting, warm and inviting"
```

---

## Antipatrones (Qué Evitar)

### ❌ 1. README.md y Documentación Extra

**NO CREAR:**
- README.md
- INSTALLATION.md
- CHANGELOG.md
- CONTRIBUTING.md
- QUICK_REFERENCE.md

**RAZÓN:** Skills son para agentes, no para humanos. Solo incluye lo que Claude necesita para hacer el trabajo.

### ❌ 2. Información "When to Use" en el Body

**INCORRECTO:**
```markdown
# SKILL.md body

## When to Use This Skill

Use this skill when users ask for image generation...
```

**CORRECTO:**
```yaml
---
description: Generate images using DALL-E. Use when users request 
image generation, logos, illustrations...
---
```

**RAZÓN:** El body solo se carga DESPUÉS de que el skill se activa. Los triggers van en `description`.

### ❌ 3. Duplicación Entre SKILL.md y References

**INCORRECTO:**

SKILL.md:
```markdown
## API Parameters
- size: "1024x1024", "1792x1024", "1024x1792"
- quality: "standard" or "hd"
- style: "vivid" or "natural"
...detailed explanation of each parameter...
```

references/api_docs.md:
```markdown
## API Parameters
- size: "1024x1024", "1792x1024", "1024x1792"
- quality: "standard" or "hd"
...same content repeated...
```

**CORRECTO:**

SKILL.md:
```markdown
## API Parameters
See references/api_docs.md for complete parameter documentation.

Quick reference:
- size: square/landscape/portrait
- quality: standard/hd
- style: vivid/natural
```

references/api_docs.md:
```markdown
## API Parameters (Complete Reference)
...detailed explanation here only...
```

### ❌ 4. Scripts Sin Testear

**NO HAGAS:**
```python
def generate_image(prompt):
    # I think this should work
    response = requests.post(...)
    # TODO: add error handling
```

**SIEMPRE:**
- Testea cada script antes de incluirlo
- Verifica que el output es el esperado
- Añade manejo de errores

### ❌ 5. SKILL.md Demasiado Largo

**LÍMITE:** ~500 líneas

**Si excedes:**
- Mueve detalles a `references/`
- Mantén solo workflow esencial
- Referencia los docs externos

### ❌ 6. Metadata de Compatibilidad Innecesaria

**NO NECESARIO (la mayoría de casos):**
```yaml
compatibility: Requires Python 3.8+, requests library
```

**SOLO SI ES CRÍTICO:**
```yaml
compatibility: Requires Windows OS, .NET Framework 4.8
```

### ❌ 7. Assets en Context Window

**INCORRECTO:**
```markdown
Read the example prompts from assets/example_prompts.txt into context
```

**CORRECTO:**
```markdown
Copy example prompts from assets/example_prompts.txt to user
```

**RAZÓN:** Assets no deben cargarse en context, se usan directamente en output.

### ❌ 8. Hardcoded Secrets

**PELIGRO:**
```python
API_KEY = "sk-1234567890abcdef"  # NUNCA HAGAS ESTO
```

**CORRECTO:**
```python
API_KEY = os.getenv("OPENAI_API_KEY")
if not API_KEY:
    raise ValueError("OPENAI_API_KEY environment variable required")
```

---

## Ejemplo Completo: Testing del Skill

### Testear con Claude

Una vez creado el skill, testéalo con consultas reales:

**Test 1: Generación Básica**
```
User: "Generate an image of a futuristic city"
Expected: Claude usa el skill, llama al script, devuelve imagen
```

**Test 2: Con Especificaciones**
```
User: "Create a landscape image, 1792x1024, high quality"
Expected: Claude pasa parámetros correctos al script
```

**Test 3: Logo Design**
```
User: "Design a minimalist logo for my startup"
Expected: Claude optimiza prompt usando mejores prácticas
```

**Test 4: Error Handling**
```
User: "Generate an image" (sin API key)
Expected: Claude detecta error y lo reporta claramente
```

### Iterar Basado en Resultados

Después de testing:

1. Nota dónde Claude tuvo problemas
2. Identifica qué faltó en SKILL.md
3. Actualiza instrucciones
4. Re-testea

---

## Conclusión

Has aprendido a crear un skill profesional para MCPs:

✅ Estructura correcta de directorios  
✅ Frontmatter YAML bien formado  
✅ Body conciso y accionable  
✅ Scripts funcionales y testeados  
✅ Referencias organizadas  
✅ Assets apropiados  
✅ Mejores prácticas aplicadas  
✅ Antipatrones evitados  

### Próximos Pasos

1. **Crea tu primer skill** siguiendo este tutorial
2. **Testéalo extensivamente** con casos reales
3. **Itera basado en feedback** de uso
4. **Comparte con la comunidad** si es útil

### Recursos Adicionales

- Consulta skills existentes como ejemplos (docx, pdf, pptx)
- Lee `references/workflows.md` para workflows complejos
- Lee `references/output-patterns.md` para patrones de calidad

---

**¡Buena suerte creando tus skills! 🚀**