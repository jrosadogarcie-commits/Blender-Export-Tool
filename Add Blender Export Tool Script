import bpy
import os
import platform
from bpy.props import StringProperty, BoolProperty, EnumProperty, IntProperty, PointerProperty, FloatVectorProperty
from bpy.types import Operator, Panel, PropertyGroup

# =============================================================
# Función auxiliar: obtener escritorio
# =============================================================
def get_desktop_path():
    if platform.system() == "Windows":
        return os.path.join(os.environ["USERPROFILE"], "Desktop")
    else:
        return os.path.join(os.path.expanduser("~"), "Desktop")

# =============================================================
# Propiedades del panel
# =============================================================
class ExportAssetSettings(PropertyGroup):
    asset_name: StringProperty(
        name="Nombre del Asset",
        description="Nombre del asset exportado. Si está vacío, se usa el nombre del objeto",
        default=""
    )
    export_path: StringProperty(
        name="Directorio de Exportación",
        description="Ruta donde se guardarán los archivos exportados",
        subtype='DIR_PATH',
        default=get_desktop_path()
    )
    export_collisions: BoolProperty(
        name="Exportar Colisiones",
        description="Incluir colisiones UCX en la exportación",
        default=True
    )
    export_lods: BoolProperty(
        name="Generar LODs",
        description="Generar LODs automáticos usando decimate modifier",
        default=True
    )
    num_lods: IntProperty(
        name="Número de LODs",
        description="Cantidad de LODs a generar",
        default=2,
        min=1,
        max=10
    )
    lod_ratios: FloatVectorProperty(
        name="Reducción LOD",
        description="Porcentaje de reducción para cada LOD (1.0 = original)",
        size=10,
        default=(0.5, 0.25, 0.125, 0.0625, 0.03125, 0.0156, 0.0078, 0.0039, 0.002, 0.001),
        min=0.0,
        max=1.0
    )
    export_format: EnumProperty(
        name="Formato",
        description="Formato de exportación del asset",
        items=[
            ("FBX", "FBX", "Exportar como FBX"),
            ("OBJ", "OBJ", "Exportar como OBJ")
        ],
        default="FBX"
    )

# =============================================================
# FUNCIONES AUXILIARES
# =============================================================
def ensure_material(obj, asset_name, operator):
    mat_name = f"M_{asset_name}"
    if not obj.data.materials:
        mat = bpy.data.materials.new(mat_name)
        obj.data.materials.append(mat)
        operator.report({'WARNING'}, f"{obj.name} no tenía material. Se creó {mat_name}.")
    else:
        mat = obj.data.materials[0]
        if not mat.name.startswith("M_"):
            mat.name = mat_name
        obj.data.materials[0] = mat

def ensure_weighted_normals(obj):
    if obj.type == "MESH":
        if not any(m.type == "WEIGHTED_NORMAL" for m in obj.modifiers):
            mod = obj.modifiers.new("WeightedNormal", 'WEIGHTED_NORMAL')
            mod.keep_sharp = True

def rename_mesh_hierarchy(obj, parent_name=""):
    if parent_name:
        new_name = f"SM_{parent_name}_{obj.name}"
    else:
        new_name = f"SM_{obj.name}"
    obj.name = new_name
    if obj.type == 'MESH':
        obj.data.name = new_name
        ensure_weighted_normals(obj)
    for child in obj.children:
        rename_mesh_hierarchy(child, new_name)

def clean_collision_mesh(col):
    """Elimina todos los modifiers y materiales de una colisión"""
    for mod in col.modifiers:
        col.modifiers.remove(mod)
    col.data.materials.clear()

def ensure_collisions(obj, operator):
    collisions = [c for c in obj.children if c.name.startswith("UCX_")]
    if not collisions:
        col = obj.copy()
        col.data = obj.data.copy()
        col.name = f"UCX_{obj.name}_01"
        col.data.name = col.name
        bpy.context.collection.objects.link(col)

        # Mantener ubicación/escala correctas al hacer hijo
        col.matrix_world = obj.matrix_world.copy()
        col.parent = obj
        col.matrix_parent_inverse = obj.matrix_world.inverted()

        col.hide_render = True
        col.display_type = 'WIRE'
        clean_collision_mesh(col)
        collisions.append(col)
        operator.report({'WARNING'}, f"No se encontraron colisiones para {obj.name}. Se creó {col.name}.")
    else:
        for i, col in enumerate(collisions, start=1):
            new_name = f"UCX_{obj.name}_{str(i).zfill(2)}"
            col.name = new_name
            if col.data:
                col.data.name = new_name
            clean_collision_mesh(col)
    return collisions

def generate_lods(obj, operator, num_lods, lod_ratios, generate_collision=True):
    lod_objs = []
    for i in range(num_lods):
        ratio = lod_ratios[i]
        lod_copy = obj.copy()
        lod_copy.data = obj.data.copy()
        bpy.context.collection.objects.link(lod_copy)

        lod_copy.matrix_world = obj.matrix_world.copy()
        lod_copy.parent = obj.parent
        if obj.parent:
            lod_copy.matrix_parent_inverse = obj.parent.matrix_world.inverted()

        mod = lod_copy.modifiers.new(f"Decimate_LOD{i+1}", 'DECIMATE')
        mod.ratio = ratio
        bpy.context.view_layer.objects.active = lod_copy
        bpy.ops.object.modifier_apply(modifier=mod.name)

        lod_copy.name = f"{obj.name}_LOD{i+1}"
        lod_copy.data.name = f"{obj.data.name}_LOD{i+1}"

        ensure_weighted_normals(lod_copy)

        if generate_collision:
            col = lod_copy.copy()
            col.data = lod_copy.data.copy()
            col.name = f"UCX_{lod_copy.name}"
            col.data.name = col.name
            bpy.context.collection.objects.link(col)

            # Mantener ubicación/escala correctas al hacer hijo
            col.matrix_world = lod_copy.matrix_world.copy()
            col.parent = lod_copy
            col.matrix_parent_inverse = lod_copy.matrix_world.inverted()

            col.hide_render = True
            col.display_type = 'WIRE'
            clean_collision_mesh(col)
            lod_objs.append(col)

        lod_objs.append(lod_copy)

    operator.report({'INFO'}, f"Generados {len(lod_objs)} objetos LOD para {obj.name}")
    return lod_objs

# =============================================================
# OPERADOR DE EXPORTACIÓN
# =============================================================
class EXPORT_OT_selected_obj(Operator):
    bl_idname = "export.selected_obj"
    bl_label = "Exportar Asset Seleccionado"
    bl_description = "Exporta el objeto con jerarquía, colisiones, LODs, materiales y weighted normals"

    def execute(self, context):
        props = context.scene.export_asset_settings
        selected_objects = context.selected_objects
        if not selected_objects:
            self.report({'ERROR'}, "No hay objetos seleccionados.")
            return {'CANCELLED'}

        for obj in selected_objects:
            if obj.parent is not None:
                continue
            if obj.type != 'MESH':
                self.report({'ERROR'}, f"{obj.name} no es un mesh válido. Saltando...")
                continue

            asset_name = props.asset_name if props.asset_name else obj.name
            export_name = f"SM_{asset_name}"

            rename_mesh_hierarchy(obj)
            ensure_material(obj, asset_name, self)

            export_objs = [obj]
            if props.export_collisions:
                collisions = ensure_collisions(obj, self)
                export_objs.extend(collisions)

            if props.export_lods:
                lods = generate_lods(obj, self, props.num_lods, props.lod_ratios, generate_collision=props.export_collisions)
                export_objs.extend(lods)

            bpy.ops.object.select_all(action='DESELECT')
            for o in export_objs:
                o.select_set(True)
            context.view_layer.objects.active = obj

            export_path = os.path.join(props.export_path, f"{export_name}.{props.export_format.lower()}")

            if props.export_format == "FBX":
                bpy.ops.export_scene.fbx(
                    filepath=export_path,
                    use_selection=True,
                    apply_unit_scale=True,
                    bake_space_transform=True,
                    mesh_smooth_type='FACE',
                    add_leaf_bones=False,
                    object_types={'MESH'},
                    use_mesh_modifiers=True,
                    use_armature_deform_only=True
                )
            elif props.export_format == "OBJ":
                bpy.ops.wm.obj_export(
                    filepath=export_path,
                    export_selected_objects=True
                )

            self.report({'INFO'}, f"Exportado correctamente: {export_path}")
        return {'FINISHED'}

# =============================================================
# OPERADOR NUEVO: APLICAR TRANSFORMACIONES
# =============================================================
class APPLY_OT_transforms(Operator):
    bl_idname = "object.apply_all_transforms"
    bl_label = "Aplicar Transformaciones"
    bl_description = "Aplica todas las transformaciones (Ctrl+A: location, rotation, scale) a los objetos seleccionados"

    def execute(self, context):
        selected_objects = context.selected_objects
        if not selected_objects:
            self.report({'ERROR'}, "No hay objetos seleccionados.")
            return {'CANCELLED'}

        for obj in selected_objects:
            bpy.context.view_layer.objects.active = obj
            bpy.ops.object.transform_apply(location=True, rotation=True, scale=True)

        self.report({'INFO'}, f"Transformaciones aplicadas a {len(selected_objects)} objeto(s).")
        return {'FINISHED'}

# =============================================================
# PANEL DE EXPORTACIÓN
# =============================================================
class EXPORT_PT_tool(Panel):
    bl_label = "Herramienta de Exportación"
    bl_idname = "EXPORT_PT_tool"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = 'Export Tool'

    def draw(self, context):
        layout = self.layout
        props = context.scene.export_asset_settings
        layout.prop(props, "asset_name")
        layout.prop(props, "export_path")
        layout.prop(props, "export_format")
        layout.prop(props, "export_collisions")
        layout.prop(props, "export_lods")
        if props.export_lods:
            layout.prop(props, "num_lods")
            for i in range(props.num_lods):
                layout.prop(props, "lod_ratios", index=i, slider=True, text=f"LOD {i+1} ratio")

        # Botón para aplicar Ctrl+A
        layout.operator(APPLY_OT_transforms.bl_idname, text="Aplicar Transformaciones (Ctrl+A)")

        layout.separator()
        layout.operator(EXPORT_OT_selected_obj.bl_idname, text="Exportar Asset")

# =============================================================
# REGISTRO
# =============================================================
classes = (
    ExportAssetSettings,
    EXPORT_OT_selected_obj,
    EXPORT_PT_tool,
    APPLY_OT_transforms,  # <-- añadido
)

def register():
    for cls in classes:
        bpy.utils.register_class(cls)
    bpy.types.Scene.export_asset_settings = PointerProperty(type=ExportAssetSettings)

def unregister():
    for cls in reversed(classes):
        bpy.utils.unregister_class(cls)
    del bpy.types.Scene.export_asset_settings

if __name__ == "__main__":
    register()
