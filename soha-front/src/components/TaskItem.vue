<template>
  <div>
    <v-layout row wrap align-center class="task-row">
      <v-checkbox hide-details class="shrink ml-2 mt-0" v-model="completed" ></v-checkbox>
        <v-text-field
          name="taskName" 
          type="text" 
          v-model="taskname"
          hide-details
          solo
          :flat="!editMode"
          :disabled="!editMode"
          @keyup.enter="updateTask"
          background-color="rgba(0, 0, 0, 0)"
        >
        </v-text-field>
          <v-menu
            bottom
            offset-y
          >
          <template v-slot:activator="{ on }">
            <v-btn
              icon
              small
              v-on="on"
            >
              <v-icon
              :color="tag.color"
              >
              {{ tag.icon }}
              </v-icon>
             </v-btn>
          </template>
          <v-list>
            <v-list-item>
              <v-chip color="default" @click="changeMission(null, '')" > 
                <v-icon>mdi-close-circle-outline</v-icon> 
                  Quitar Misión 
                </v-chip>
            </v-list-item>
            <v-list-item :id="element._id" v-for="(element) in missionList" 
                    :key="element._id" >
              <v-chip :color="element.displayColor" dark @click="changeMission(element._id, element.displayColor)" > {{ element.missionName }}</v-chip>
            </v-list-item>
          </v-list>
        </v-menu>

        <v-list-item-avatar v-if="delegated&&!allowEdit">
          <v-img :src="avatar"></v-img>
        </v-list-item-avatar>

        <v-speed-dial
        right
        direction="right"
        open-on-hover

        transition="slide-x"
        >
          <template v-slot:activator v-if="allowEdit">
            <v-btn
            icon
            @click="editTask"
            >
            <v-icon>edit</v-icon>
            </v-btn>
          </template>
          <v-btn
              fab
              dark
              small
              color="red"
              @click="deleteTask"
          >
            <v-icon>delete</v-icon>
          </v-btn>
        </v-speed-dial>
      </v-layout>
    </div>
</template>

<script>
import axios from "axios";

export default {
  name: "TaskItem",
  props: {
    taskname: String,
    completed: Boolean,
    allowEdit: Boolean,
    taskId: String,
    missionTags: Array,
    delegated: Boolean,
    assignedTo: Array
  },
  data() {
    return {
      editMode: false,
      tag: {icon: 'bookmark', color: 'blue'},
      missionList: [],
      avatar: ''
    }
  },
  mounted() {
    this.renderMissionTag()
    this.getMyMissions()
    this.renderAvatar()
  },
  computed: {
    updateColor() {
      this.renderMissionTag()
    },
    updateAvatar() {
      this.renderAvatar()
    },
    udpdateAvailableMissions() {
      this.getMyMissions()
    }
  },
  methods: {
    editTask() {
        this.editMode = !this.editMode
    },
    updateTask() {
      // TODO: Read user ID off the current session
      const currentUser = '5d46632ebfbbe11ab5f5e5f0'

      axios
        .put("http://localhost:3000/"+currentUser+"/"+this.taskId+"/changeTaskName", {
          name: this.taskname, 
          completed: this.completed 
        })
        .then(res => {
          // Disable edit mode
          this.editMode = false

        })
        .catch(err => {
          alert(
            "Lo sentimos, no se pudo modificar la tarea, favor de intentar más tarde.", err
          );
        }); 
    },
    deleteTask() {
      // TODO: Implementar ruta para borrar task
      // TODO: Usar splice para eliminar el índice en específico
      // Creo que necesito invocar el método del componente padre
    },
    changeMission(missionId, newColor) {
        console.log("Changing Mission")
        // TODO: Read user ID off the current session
        const currentUser = '5d46632ebfbbe11ab5f5e5f0'

        const secondUrl = "http://localhost:3000/"+currentUser+"/"+this.taskId+"/assignMission"
        console.log(secondUrl)
        axios
          .put(secondUrl, {
            missionId: missionId 
          })
          .then(res => {
            this.missionTags = res.data
            this.tag.color = newColor
            this.tag.icon = 'bookmark'
          })
          .catch(err => {
            alert(
              "Lo sentimos, hubo un problema al modificar la misión (reasignar). Inténtalo más tarde", err
            );
        });

    },
    renderMissionTag() {
      if ( this.missionTags === null || this.missionTags.length === 0 ){
        this.tag.color = ''
        this.tag.icon = 'mdi-bookmark-outline'
      } else {
        this.tag.color = this.missionTags[0].displayColor
        this.tag.icon = 'bookmark'
      }

    },
    renderAvatar() {
      if ( this.assignedTo === null || this.assignedTo.length === 0 ){
        this.avatar = ''
      } else {
        this.avatar = this.assignedTo[0].profilePic
      }

    },
    getMyMissions() {
      // Get my tasks from myTasks route, then render them to
      // this.kpiList[]
      // TODO: Read user ID off the current session
      const currentUser = '5d46632ebfbbe11ab5f5e5f0'

      const url = "http://localhost:3000/"+currentUser+"/myMissions"
      axios
        .get(url)
        .then(res => {
          this.missionList = res.data
          // console.log("My Mission List",this.missionList)
        })
        .catch(err => {
          alert(
            "Lo sentimos, hubo un problema al traer tus misiones. Inténtalo más tarde", err
          );
        });
    },
  }
}
</script>
<style scoped lang="scss">
    .task-row {
        padding: 0;
        cursor: move;
        &:hover {
            background: lightgrey;
        }
    }
</style>
