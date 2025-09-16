# Blender Export Tool

[![Blender Version](https://img.shields.io/badge/Blender-3.x%2B-blue)](https://www.blender.org/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

**Blender Export Tool** es un complemento para Blender que facilita la exportación de assets con jerarquía, colisiones, LODs, materiales y transformaciones aplicadas. Ideal para artistas y desarrolladores que importan modelos de otros softwares y quieren exportarlos a FBX u OBJ sin problemas de escala, posición o materiales.

---

## 🔹 Características

- Exportación de objetos seleccionados con **jerarquía completa**.
- Creación automática de **colisiones UCX** si no existen.
- Generación automática de **LODs** con decimate modifier.
- Aplicación de **Weighted Normals** a los meshes.
- Asignación automática de **materiales** si no existen.
- Botón para **aplicar todas las transformaciones (Ctrl+A)** en los objetos seleccionados.
- Exportación en **FBX** y **OBJ**.
- Configuración flexible de nombres, rutas, número de LODs y ratios de reducción.

---

## 🔹 Instalación

1. Descargar el archivo `export_tool.py`.
2. Abrir Blender → `Edit → Preferences → Add-ons → Install`.
3. Seleccionar el archivo `.py` y activar el addon.
4. Encontrarás la herramienta en el panel lateral **N → Export Tool**.

---

## 🔹 Uso

1. Selecciona los objetos que deseas exportar.
2. Presiona el botón **“Aplicar Transformaciones (Ctrl+A)”** si el objeto viene de otro software. Esto aplica ubicación, rotación y escala.
3. Configura el nombre del asset, ruta de exportación, colisiones, LODs y formato.
4. Pulsa **“Exportar Asset”** para generar los archivos.

---

## 🔹 Flujo de trabajo recomendado

1. Importar modelo desde otro software.
2. Seleccionar todos los objetos.
3. Presionar **Aplicar Transformaciones**.
4. Configurar opciones de exportación y LODs.
5. Exportar como FBX o OBJ.

---

## 🔹 Requisitos

- Blender 3.x o superior.

---

## 🔹 Licencia

MIT License.  
Puedes usar y modificar este addon libremente, pero no nos hacemos responsables de daños o pérdidas de datos.

---

## 🔹 Capturas de pantalla (opcional)

_Se recomienda agregar capturas del panel, botones y flujo de exportación aquí._

---

## 🔹 Contribuciones

Las contribuciones son bienvenidas. Puedes abrir issues, sugerir mejoras o enviar pull requests.

