<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-card>
          <v-card-title class="headline">Guest List</v-card-title>
          <v-card-text>
            <v-data-table
              :headers="headers"
              :items="guests"
              :search="search"
              class="elevation-1"
            >
              <template v-slot:top>
                <v-toolbar flat>
                  <v-toolbar-title>Guests</v-toolbar-title>
                  <v-divider
                    class="mx-4"
                    inset
                    vertical
                  ></v-divider>
                  <v-spacer></v-spacer>
                  <v-btn @click="openAddGuestModal" color="primary">Add Guest</v-btn>
                </v-toolbar>
              </template>
              <template v-slot:item.actions="{ item }">
                <v-icon
                  small
                  @click="editGuest(item)"
                  class="mr-2"
                >
                  mdi-pencil
                </v-icon>
                <v-icon
                  small
                  @click="deleteGuest(item)"
                >
                  mdi-delete
                </v-icon>
              </template>
            </v-data-table>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <div>Total Guests: {{ totalGuests }}</div>
          </v-card-actions>
        </v-card>

        <!-- Add Guest Modal -->
        <v-dialog v-model="showAddGuestModal" max-width="600">
          <v-card>
            <v-card-title class="headline">{{ editingIndex > -1 ? 'Edit Guest' : 'Add Guest' }}</v-card-title>
            <v-card-text>
              <v-form @submit.prevent="addGuest">
                <v-text-field v-model="newGuest.email" label="Email"></v-text-field>
                <v-text-field v-model="newGuest.tickets" label="Tickets" type="number"></v-text-field>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn type="submit" color="primary">{{ editingIndex > -1 ? 'Save Changes' : 'Add Guest' }}</v-btn>
                  <v-btn @click="cancelAddGuest">Cancel</v-btn>
                </v-card-actions>
              </v-form>
            </v-card-text>
          </v-card>
        </v-dialog>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import GuestRepository from 'src/services/guest-repository';

export default {
  data() {
    return {
      guests: [],
      showAddGuestModal: false,
      newGuest: {
        email: '',
        tickets: 0,
      },
      editingIndex: -1,
      search: '',
      headers: [
        { text: 'Email', value: 'email' },
        { text: 'Tickets', value: 'tickets' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
    };
  },
  async mounted() {
    this.guests = await new GuestRepository().load();
  },
  computed: {
    totalGuests() {
      return this.guests.reduce((total, guest) => total + guest.tickets, 0);
    },
  },
  methods: {
    editGuest(item) {
      const index = this.guests.indexOf(item);
      this.editingIndex = index;
      this.newGuest = { ...item };
      this.showAddGuestModal = true;
    },
    deleteGuest(item) {
      const index = this.guests.indexOf(item);
      this.guests.splice(index, 1);
      this.saveGuests();
    },
    openAddGuestModal() {
      this.newGuest = { email: '', tickets: 0 };
      this.showAddGuestModal = true;
    },
    addGuest() {
      if (this.editingIndex > -1) {
        // Editing existing guest
        this.guests.splice(this.editingIndex, 1, this.newGuest);
        this.editingIndex = -1;
      } else {
        // Adding new guest
        this.guests.push(this.newGuest);
      }
      this.saveGuests();
      this.showAddGuestModal = false;
    },
    cancelAddGuest() {
      this.showAddGuestModal = false;
      this.editingIndex = -1;
    },
    async saveGuests() {
      await new GuestRepository().save(this.guests);
    },
  },
};
</script>

<style scoped>
/* Add your custom styles here if needed */
</style>
