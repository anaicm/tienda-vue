<template>
  <div>
    <div>
      <h2>portal de mantenimiento de usuarios</h2>
      <div class="progress" v-show="isLoading">
        <!--Se va a mostrar cuando este cargando los datos-->
        <div class="indeterminate"></div>
      </div>
      <!-- Mostrar la tabla si hay usuarios -->
      <table v-if="usuarios.length > 0" v-show="!isLoading">
        <!--Se muestra cuando no este cargando los datos-->
        <thead>
          <tr>
            <th>ID</th>
            <th>Username</th>
            <th>Email</th>
            <!-- Agrega más columnas según las propiedades de tus usuarios -->
          </tr>
        </thead>
        <tbody>
          <tr v-for="usuario in usuarios" :key="usuario.id">
            <td>{{ usuario.id }}</td>
            <td>{{ usuario.userName }}</td>
            <td>{{ usuario.email }}</td>
            <td class="center" @click="abrirModalModificar(usuario)">
              <i class="material-icons">edit</i>
            </td>
            <td class="center" @click="abrirPortalEstablecimineto('portalCliente', usuario)">
                <i class="material-icons" style="margin-left: 10px">event_note</i>
            </td>
            <td class="center" @click="eliminarUsuario(usuario.id)">
              <i class="material-icons">delete</i>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Mostrar mensaje si no hay usuarios -->
      <div v-else>No hay usuarios para mostrar.</div>
    </div>
    <!--Modal actualizar---------------------------------------->
    <div id="modalModificar" class="modal">
      <div class="modal-content">
        <!-- Inputs para la edición de usuario -->
        <label for="username">Username:</label>
        <input
          type="text"
          id="username"
          v-model="usuarioSeleccionado.userName"
        />

        <label for="email">Email:</label>
        <input type="text" id="email" v-model="usuarioSeleccionado.email" />
      </div>
      <div class="modal-footer">
        <!-- Botones para cerrar y guardar cambios -->
        <a class="modal-close waves-effect waves-green btn-flat">Cerrar</a>
        <a
          class="modal-close waves-effect waves-green btn-flat"
          @click="guardarCambios"
          >Guardar Cambios</a
        >
      </div>
    </div>
  </div>
</template>

<script>
import M from "materialize-css";
export default {
  name: "portalUsuario",
  data() {
    return {
      usuarios: [],
      isLoading: false, // se inicializa en false para la carga
      usuarioSeleccionado: {
        id: -1,
        userName: "",
        email: "",
      },
    };
  },
  methods: {
    abrirModalModificar(usuario) {
      // Abrir el modal de edición y asignar el usuario seleccionado
      this.usuarioSeleccionado = {
        id: usuario.id,
        userName: usuario.userName,
        email: usuario.email,
      };
      const modal = M.Modal.init(document.getElementById("modalModificar"));
      modal.open();
    },
    abrirPortalEstablecimineto(componente, usuario) {//función generica para cuando hace click que navege al componente
      // Redireccionar al componente especificado
      this.$store.commit("setNameUsuarioSeleccionado", usuario.userName);
      this.$store.commit("setAdminUsuarioSeleccionado", usuario.id);//guarda el id del usuario que hace clic
      this.$router.push(`/${componente}`);
    },

    async guardarCambios() {
      // Realizar la solicitud para actualizar el usuario
      try {
        const response = await this.axios.put(
          `/Users/${this.usuarioSeleccionado.id}`,
          {
            username: this.usuarioSeleccionado.userName,
            email: this.usuarioSeleccionado.email,
          }
        );
        console.log("Usuario actualizado con éxito:", response);
        // Buscar y actualizar el usuario en la lista existente
        const index = this.usuarios.findIndex(
          (user) => user.id === this.usuarioSeleccionado.id
        );
        if (index !== -1) {
          // Actualizar el usuario en la lista
          this.usuarios[index] = {
            id: this.usuarioSeleccionado.id,
            userName: this.usuarioSeleccionado.userName,
            email: this.usuarioSeleccionado.email,
          };
        }
      } catch (error) {
        console.error("Error al actualizar usuario:", error);
      }
    },
    async eliminarUsuario(idUsuario) {
      const confirmacion = window.confirm(
        "¿Estás seguro de que deseas eliminar este usuario?"
      );
      if (!confirmacion) {
        // El usuario canceló la eliminación
        return;
      }
      // Realizar la solicitud para eliminar el usuario por ID
      try {
        const response = await this.axios.delete(`/Users/${idUsuario}`);
        console.log("Usuario eliminado con éxito:", response);

        // Filtrar la lista para quitar el usuario eliminado
        this.usuarios = this.usuarios.filter((user) => idUsuario !== user.id);
      } catch (error) {
        console.error("Error al eliminar usuario:", error);
      }
    },
  },

  async mounted() {
    // Realizar la solicitud a la API para obtener los usuarios
    try {
      this.isLoading = true; //cuando empieza a cargar los datos se pone en true
      const response = await this.axios.get("/Users", {
        withCredentials: true,
      });
      // Asignar los usuarios a la propiedad 'usuarios'
      this.usuarios = response.data;
      //para la paginación
    } catch (error) {
      console.error("Error al obtener usuarios:", error);
    }
    this.isLoading = false; //cuando finaliza vuelve a estado inicial de false
  },
};
</script>
