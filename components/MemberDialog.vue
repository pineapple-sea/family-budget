<template>
    <v-dialog v-model="show" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">User Profile</span>
        </v-card-title>
        <v-card-text>
          <v-row>
            <v-col cols="12" md="6">
              <v-text-field v-model="firstName" label="First name" required color="orange"></v-text-field>
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field v-model="lastName" label="Last name" required color="orange">
              </v-text-field>
            </v-col>
            <v-col cols="12">
              <v-text-field
                v-model="salary"
                label="Salary"
                hint="Input your month salary"
                required
                color="orange"
              ></v-text-field>
              </v-col>
            </v-row>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn v-if="member" color="orange darken-1" text @click="deleteMember">Delete</v-btn>
          <v-btn color="orange darken-1" text @click="closeDialog">Close</v-btn>
          <v-btn color="orange darken-1" text @click="save">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
</template>

<script>
export default {
	props: {
		value: {
      type: Boolean
    },
    mode: {
      type: String
    },
    member: {
      default: {}
    }
	},
  data() {
    return {
      firstName: '',
      lastName: '',
      salary: 0,
    }
  },
  methods: {
    clearForm() {
      this.firstName = '';
      this.lastName = '';
      this.salary = '';
    },
    closeDialog() {
      this.show = false;

      if (this.mode === 'add') {
        this.clearForm();
      }
    },
    save() {
      if (this.mode === 'add') {
        this.$emit('addMember', {
          firstName: this.firstName,
          lastName: this.lastName,
          salary: this.salary,
        });
      }
      
      if (this.mode === 'edit') {
        this.$emit('editMember', {
          firstName: this.firstName,
          lastName: this.lastName,
          salary: this.salary,
          id: this.member.id
        });
      }

      this.closeDialog();
    },
    deleteMember() {
      this.$emit('deleteMember', this.member.id);

      this.closeDialog();
    },
  },
	computed: {
		show: {
			get () {
				return this.value
			},
			set (value) {
				this.$emit('input', value)
			}
		}
	},
  watch: {
    member({firstName, lastName, salary}) {
      this.firstName = firstName;
      this.lastName = lastName;
      this.salary = salary;
    }
  }
}
</script>
