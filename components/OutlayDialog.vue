<template>
    <v-dialog v-model="show" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Cost List</span>
        </v-card-title>
        <v-card-text>
          <v-row v-if="mode === 'add'">
            <v-col cols="12">
              <v-text-field v-model="costType" label="Cost type" required color="orange"></v-text-field>
            </v-col>
            <v-col cols="12">
              <v-text-field v-model="costName" label="Cost name" required color="orange"></v-text-field>
            </v-col>
            <v-col cols="12">
              <v-text-field
                v-model="cost"
                label="Cost"
                hint="Input your month costs"
                required
                color="orange"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row v-else>
            <v-card v-for="cost of outlays" :key="cost.id">
              <v-row>
                <v-col cols="6">
                  <v-text-field v-model="cost.costName" label="Cost name" required color="orange"></v-text-field>
                </v-col>

                <v-col cols="6">
                  <v-text-field
                    v-model="cost.cost"
                    label="Cost"
                    hint="Input your month outlays"
                    required
                    color="orange"
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-card>
          </v-row>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn v-if="outlays" color="orange darken-1" text @click="deleteOutlays">Delete</v-btn>
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
    outlays: {
      default: {}
    }
	},
  data() {
    return {
      costType: '',
      costName: '',
      cost: 0,
    }
  },
  methods: {
    clearForm() {
      this.costType = '';
      this.costName = '';
      this.cost = '';
    },
    closeDialog() {
      this.show = false;

      if (this.mode === 'add') {
        this.clearForm();
      }
    },
    save() {
      if (this.mode === 'add') {
        this.$emit('addOutlay', {
          costType: this.costType,
          costName: this.costName,
          cost: this.cost,
        });
      }
      
      if (this.mode === 'edit') {
        this.$emit('editOutlay', this.outlays);
      }

      this.closeDialog();
    },
    deleteOutlays() {
      this.$emit('deleteOutlays', this.outlays);

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
}
</script>
