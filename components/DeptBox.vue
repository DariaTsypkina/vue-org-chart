<template lang='pug'>
div(
  v-on:mouseenter='mouseOverBox(true)',
  v-on:mouseleave='mouseOverBox(false)'
)
  template(v-if='departmentData')
    .department
      .col(
        :id='"ID_" + departmentData.id',
        :class='[type, active, managerPhoto]',
        @click='setActiveDepartment(departmentData, $event)',
        @touchend='setActiveDepartment(departmentData, $event)',
        v-on:contextmenu.prevent='showCtxMenu(departmentData, $event)'
      ) 
        table
          tr
            td.ppl_count0
              .ppl_count(v-if='showNrPeople')
                .ppl_count_nr(
                  v-if='departmentData.employees.length',
                  :title='departmentData.employees.length + (departmentData.employees.length === 1 ? " person in this department" : " people in this department")'
                ) {{ departmentData.employees.length }}
            td(v-if='managerPhotoView')
              img.profile(
                :src='config.photoUrl.prefix + departmentData.manager.photo + config.photoUrl.suffix',
                v-if='departmentData.manager.photo'
              )
              .material-icons.nophoto(v-else) face
            td
              .level_indicator(
                :style='{ backgroundColor: config.levelColors[level - 1] || "#FFFFFF" }'
              )
              .textdiv(
                :style='{ height: config.boxHeight + "px", width: config.boxWidth + "px" }'
              )
                .name(v-html='departmentData.name')
                .name_manager(v-if='managerNameView') {{ departmentData.manager.name }}
            td.drill0
              .drill 
                template(v-if='departmentData.children.length')
                  i.material-icons.arrow.down(
                    v-if='!departmentData.showChildren',
                    @click.prevent='doShowChildren(true)',
                    @touchend.prevent='doShowChildren(true)'
                  ) arrow_drop_down
                  i.material-icons.arrow.up(
                    v-if='departmentData.showChildren',
                    @click.prevent='doShowChildren(false)',
                    @touchend.prevent='doShowChildren(false)'
                  ) arrow_drop_up
                  template(v-if='showNrDepartments')
                    .hidden_dept.down(
                      v-if='!departmentData.showChildren',
                      @click.prevent='doShowChildren(true)',
                      @touchend.prevent='doShowChildren(true)',
                      :title='departmentData.children.length + " subdepartment" + (departmentData.children.length === 1 ? "" : "s")'
                    ) {{ departmentData.children.length }}
                    .hidden_dept.up(
                      v-if='departmentData.showChildren',
                      @click.prevent='doShowChildren(false)',
                      @touchend.prevent='doShowChildren(false)',
                      :title='departmentData.children.length + " subdepartment" + (departmentData.children.length === 1 ? "" : "s")'
                    ) {{ departmentData.children.length }}

      i.material-icons.view_button(
        v-if='displaySiblingIcon',
        v-on:click='showViewMenu(departmentData, $event)',
        title='Show/hide parents'
      ) visibility
      i.material-icons.hidden_parents(
        v-if='hiddenParents',
        v-on:click='setHideParents(false)',
        title='Show parents'
      ) more_vert
    template(v-if='!departmentData')
      .department.invisible(
        v-if='!managerPhotoView',
        :class='[type]'
      )
      .department.manager_photo.invisible(v-else, :class='[type]')
</template>

<script>
import { mapState, mapActions } from 'vuex'

export default {
  name: 'DeptBox',
  props: {
    departmentData: {
      type: Object,
      default: null,
    },
    level: {
      type: Number,
      required: true,
      default: 0,
    },
    type: {
      type: String,
      default: '',
    },
  },
  data: function () {
    return {
      displaySiblingIcon: false,
      hiddenDept: 5,
    }
  },
  computed: {
    ...mapState([
      'managerNameView',
      'managerPhotoView',
      'activeDepartment',
      'editMode',
      'config',
      'chart',
      'showNrPeople',
      'showNrDepartments',
    ]),
    managerPhoto: function () {
      return this.managerPhotoView ? 'manager_photo' : ''
    },
    active: function () {
      return this.departmentData === this.activeDepartment
        ? 'active_department'
        : ''
    },
    hiddenParents: function () {
      return this.departmentData === this.chart && this.chart.parent
    },
  },
  methods: {
    ...mapActions([
      'showChildren',
      'hideChildren',
      'setHideSiblings',
      'setHideParents',
      'toggleHideParents',
    ]),

    doShowChildren(down) {
      var department =
        this.departmentData.parent &&
        !this.departmentData.parent.showChildren &&
        down
          ? this.departmentData.parent
          : this.departmentData
      if (down) {
        // debugger
        this.showChildren(department)
      } else {
        this.hideChildren(department)
      }
    },
    setActiveDepartment(department, event) {
      this.$store.commit('setActiveDepartment', department)
    },
    showCtxMenu(department, event) {
      this.$store.commit('setActiveDepartment', department)
      this.$store.commit('showEditMenu', null)

      this.$nextTick((e) => {
        if (this.editMode) {
          this.$store.commit('showEditMenu', event)
        }
      })
    },
    showViewMenu(department, event) {
      this.$store.commit('setActiveDepartment', department)
      this.toggleHideParents()

      /*
      this.$store.commit('setActiveDepartment', department)
      this.$store.commit('showViewMenu', null)
      
      this.$nextTick(e => {
        this.$store.commit('showViewMenu', event)
      })
      */
    },
    mouseOverBox(value) {
      if (!value) {
        this.displaySiblingIcon = false
      } else {
        if (
          this.departmentData.parent //&& this.departmentData.parent.showChildren
        ) {
          this.displaySiblingIcon = true
        }
      }
    },
    hideSiblings() {
      this.setHideSiblings(this.departmentData)
    },
  },

  // mounted: function () {
  //   console.log(
  //     this.departmentData.name,
  //     this.departmentData.showChildren
  //   )
  // },
}
</script>
<style scoped>
.down-icon1 {
  width: 20px;
  height: auto;
  margin: 3px 0px 0px 0px;
}
.profile {
  width: 55px;
  max-height: 55px;
  display: block;
  margin: auto;
  border-radius: 30px;
}
.active_department {
  background-color: yellow !important;
  color: black !important;
}
.arrow {
  font-size: 30px;
  bottom: 0px;
  right: 0px;
  margin: -8px;
}
.down {
  cursor: pointer;
}
.up {
  cursor: pointer;
}
.down:hover,
.up:hover {
  color: red;
}
.view_button {
  font-size: 24px;
  position: absolute;
  top: 0px;
  right: 0px;
  margin: 0px;
  color: black;
}
.drill {
  width: 10px;
  height: 100%;
  position: absolute;
  bottom: 0px;
}
.drill0,
.ppl_count0 {
  position: relative;
  height: 100%;
}
.ppl_count {
  height: 100%;
  color: grey;
  font-size: 12px;
}
.ppl_count_nr {
  position: absolute;
  bottom: 0px;
  left: -10px;
  color: grey;
  font-size: 12px;
  background-color: white;
  padding: 0px 2px;
}
.department {
  border: 0px solid rgb(180, 180, 180);
  /* margin: 30px 0px 5px 0px; */
  text-align: center;
  font-size: 11px;
  vertical-align: middle;
  display: flex;
  border-radius: 3px;
  align-items: center;
  cursor: pointer;
  box-sizing: border-box;
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  background-color: white;
  margin-left: auto;
  margin-right: auto;
  padding: 2px 2px;
  position: relative;
  width: 100%;
  /*box-shadow: 3px 3px 3px lightgrey;*/
}
.manager_photo {
  margin-top: 20px;
}
.invisible {
  visibility: hidden;
}
.level_indicator {
  border-radius: 5px;
  height: 5px;
  margin-bottom: 5px;
}

.column {
  margin-top: 1px;
  margin-bottom: 0px;
}
.staff,
.staff_column {
  margin: 2px 80px 2px 80px;
}
.staff_child {
  margin: 2px 80px 2px 100px;
}
.name,
.name1,
.name2 {
  overflow-wrap: break-word;
  min-width: 1%;
  min-height: 1.9rem;
}
.name1 {
  top: 10px;
}
.name2 {
  top: 14px;
}
.name_manager {
  overflow-wrap: break-word;
  min-width: 1%;
  display: inline-block;
  color: grey;
}
.hidden_dept {
  bottom: 10px;
  right: 1px;
  width: 14px;
  color: grey;
  font-size: 12px;
  padding: 0px 2px;
  border-radius: 4px;
}

.hidden_parents,
.hidden_parents1 {
  position: absolute;
  top: -24px;
  left: 78px;
  font-size: 24px;
  color: grey;
}
.hidden_parents1 {
  left: 50px;
}
.nophoto {
  font-size: 52px;
  color: lightgrey;
}
.col {
  border: 1px solid grey;
  border-collapse: collapse;
  margin: auto;
  padding: 5px 10px;
  border-radius: 3px;
}
.material-icons.arrow {
  position: absolute;
  bottom: -5px;
}
.hidden_dept {
  position: absolute;
  bottom: 5px;
}
</style>
