<template>
  <v-container class="about">
    <v-row class="mt-7">
      <v-col><h1 class="text-center">Empleados</h1></v-col>
    </v-row>
    <v-row>
      <v-col offset="2" cols="8">
        <v-row class="mt-16 mb-3">
          <v-col offset="11" cols="1" class="text-right">
            <v-btn small fab color="primary" @click="nuevoEmpleado">
              <v-icon width="13" height="13">fa-solid fa-plus</v-icon>
            </v-btn>
          </v-col>
        </v-row>
        <template>
          <v-card color="accent">
            <v-card-title>
              <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Search"
                single-line
                hide-details
              ></v-text-field>
            </v-card-title>
            <v-data-table
              :headers="headers"
              :items="empleados"
              :search="search"
              class="elevation-0"
            >
              <template #item.opciones="{ item }">
                <!-- OPCIONES Editar / Detalles / Eliminar -->
                <v-tooltip bottom color="primary">
                  <template #activator="{ on }">
                    <v-btn
                      text
                      small
                      fab
                      color="amber darken-3"
                      v-on="on"
                      @click="editarEmpleado(item)"
                    >
                      <v-icon>fa-pen</v-icon>
                    </v-btn>
                  </template>
                  <span>Editar</span>
                </v-tooltip>

                <v-tooltip bottom color="primary">
                  <template #activator="{ on }">
                    <v-btn
                      text
                      small
                      fab
                      color="orange darken-3"
                      v-on="on"
                      @click="showDetail(item)"
                    >
                      <v-icon>fa-bars</v-icon>
                    </v-btn>
                  </template>
                  <span>Detalles</span>
                </v-tooltip>

                <v-tooltip bottom color="primary">
                  <template #activator="{ on }">
                    <v-btn
                      text
                      small
                      fab
                      color="primary"
                      v-on="on"
                      @click="showDeleteDialog(item)"
                    >
                      <v-icon>fa-trash-alt</v-icon>
                    </v-btn>
                  </template>
                  <span>Eliminar</span>
                </v-tooltip>
              </template>
            </v-data-table>
          </v-card>
        </template>
      </v-col>
    </v-row>
    <!-- Model Editar -->
    <template>
      <v-row justify="center">
        <v-dialog v-model="dialogEditar" persistent max-width="600px">
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ title }}</span>
            </v-card-title>
            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="4">
                    <v-text-field
                      label="Documento"
                      v-model="empleado.documento_identidad"
                      required
                      :disabled="readOnly"
                    ></v-text-field>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12" sm="6">
                    <v-text-field
                      label="Nombres"
                      v-model="empleado.nombres"
                      required
                      :disabled="readOnly"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-text-field
                      label="Apellidos"
                      persistent-hint
                      v-model="empleado.apellidos"
                      required
                      :disabled="readOnly"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-autocomplete
                      v-model="selectArea"
                      :items="areas"
                      item-text="nombre"
                      item-value="area_id"
                      label="Area"
                      required
                      :disabled="readOnly"
                    ></v-autocomplete>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-autocomplete
                      v-model="selectSubArea"
                      :items="selectAreaFilter"
                      item-text="nombre"
                      item-value="subarea_id"
                      label="Sub areas"
                      :disabled="readOnly"
                    ></v-autocomplete>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="dialogEditar = false">
                Close
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="putEmpleado"
                v-if="!readOnly"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-row>
    </template>
    <!-- Model delte -->
    <template>
      <v-row justify="center">
        <v-dialog v-model="dialogDelete" max-width="400">
          <v-card>
            <v-card-title class="text-h5"> Eliminar empleado </v-card-title>

            <v-card-text class="text-subtitle font-weight-medium"
              >¿ Esta seguro de borrar este empleado ?
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>

              <v-btn
                color="primary darken-1"
                text
                @click="dialogDelete = false"
              >
                No
              </v-btn>

              <v-btn color="red darken-1" text @click="deleteEmpleado">
                Si
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-row>
    </template>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      dialogEditar: false,
      dialogDelete: false,
      search: "",
      readOnly: false,
      empleados: [],
      areas: [],
      subAreas: [],
      selectAreaFilter: [],
      selectArea: 0,
      selectSubArea: 0,
      empleadoId: 0,
      isNew: false,
      title: "",
      empleado: {
        documento_identidad: "",
        nombres: "",
        apellidos: "",
        subarea: "",
      },
      headers: [
        {
          text: "N° Documento",
          value: "documento_identidad",
          width: "160",
          class: "titulo",
        },
        {
          text: "Nombres",
          value: "nombres",
          width: "160",
          class: "titulo",
        },
        {
          text: "Apellidos",
          value: "apellidos",
          width: "160",
          class: "titulo",
        },
        {
          text: "N° Documento",
          value: "subarea_nombre",
          width: "160",
          class: "titulo",
        },
        {
          text: "Opciones",
          value: "opciones",
          sortable: false,
          width: "120",
          align: "right",
          class: "titulo",
        },
      ],
    };
  },
  watch: {
    selectArea(value) {
      console.log(value);
      this.selectAreaFilter = this.subAreas.filter((m) => m.area == value);
    },
  },
  methods: {
    getEmpleados() {
      axios.get("http://127.0.0.1:8000/api/empleados/").then((response) => {
        this.empleados = response.data;
        console.log(response.data);
      });
    },
    getAreas() {
      axios.get("http://127.0.0.1:8000/api/areas/").then((response) => {
        this.areas = response.data;
        console.log(response.data);
      });
    },
    getSubAreas() {
      axios.get("http://127.0.0.1:8000/api/subareas/").then((response) => {
        this.subAreas = response.data;
        console.log(response.data);
      });
    },
    editarEmpleado(item) {
      this.empleado = {
        documento_identidad: item.documento_identidad,
        nombres: item.nombres,
        apellidos: item.apellidos,
        subarea: item.subarea,
      };

      this.selectArea = item.area;
      this.selectSubArea = item.subarea;
      this.empleadoId = item.empleado_id;
      this.title = "Editar Empleado";
      this.dialogEditar = true;
    },
    putEmpleado() {
      let newEmpleado = {
        documento_identidad: this.empleado.documento_identidad,
        nombres: this.empleado.nombres,
        apellidos: this.empleado.apellidos,
        subarea: this.selectSubArea,
      };
      console.log(newEmpleado);
      if (this.isNew) {
        axios
          .post(`http://127.0.0.1:8000/api/empleados/`, newEmpleado)
          .then((response) => {
            console.log("Empleado actualizado:", response.data);
            this.getEmpleados();
          })
          .catch((error) => {
            console.error("Error al actualizar el empleado:", error);
          });
      } else {
        this.empleado = {
          documento_identidad: "",
          nombres: "",
          apellidos: "",
          subarea: 0,
        };
        axios
          .put(
            `http://127.0.0.1:8000/api/empleados/${this.empleadoId}/`,
            newEmpleado
          )
          .then((response) => {
            console.log("Empleado actualizado:", response.data);
            this.getEmpleados();
          })
          .catch((error) => {
            console.error("Error al actualizar el empleado:", error);
          });
      }

      this.dialogEditar = false;
    },
    showDetail(item) {
      this.empleado = {
        documento_identidad: item.documento_identidad,
        nombres: item.nombres,
        apellidos: item.apellidos,
        subarea: item.subarea,
      };

      this.selectArea = item.area;
      this.selectSubArea = item.subarea;
      this.empleadoId = item.empleado_id;

      this.dialogEditar = true;
      this.readOnly = true;
      this.title = "Detalle Empleado";
    },
    showDeleteDialog(item) {
      this.empleadoId = item.empleado_id;
      this.dialogDelete = true;
    },
    deleteEmpleado() {
      axios
        .delete(`http://127.0.0.1:8000/api/empleados/${this.empleadoId}/`)
        .then((response) => {
          console.log("Empleado deleteado:", response.data);
          this.getEmpleados();
        })
        .catch((error) => {
          console.error("Error al borrar el empleado:", error);
        });
      this.dialogDelete = false;
    },
    nuevoEmpleado() {
      this.empleado = {
        documento_identidad: "",
        nombres: "",
        apellidos: "",
        subarea: 0,
      };

      this.dialogEditar = true;
      this.isNew = true;
      this.readOnly = false;
    },
  },
  created() {
    this.getEmpleados();
    this.getAreas();
    this.getSubAreas();
  },
};
</script>

<style lang="scss" scoped></style>
