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
                <v-btn @click="editGuest(item)" color="secondary" variant="tonal">Edit</v-btn>
                <v-btn @click="deleteGuest(item)" color="red-lighten-2" variant="tonal">Delete</v-btn>
              </template>
            </v-data-table>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <div>Total Guests: {{ totalGuests }}</div>
          </v-card-actions>
        </v-card>

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

<script lang="ts">
import { defineComponent } from 'vue';
import GuestRepository from 'src/services/guest-repository';

interface Guest {
  email: string;
  tickets: number;
}

export default defineComponent({
  data() {
    return {
      guests: [] as Guest[],
      showAddGuestModal: false,
      newGuest: {
        email: '',
        tickets: 1,
      } as Guest,
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
    totalGuests(): number {
      return this.guests.reduce((total, guest) => total + Number(guest.tickets), 0);
    },
  },
  methods: {
    editGuest(item: Guest): void {
      const index = this.guests.indexOf(item);
      this.editingIndex = index;
      this.newGuest = { ...item };
      this.showAddGuestModal = true;
    },
    deleteGuest(item: Guest): void {
      const index = this.guests.indexOf(item);
      this.guests.splice(index, 1);
      this.saveGuests();
    },
    openAddGuestModal(): void {
      this.newGuest = { email: '', tickets: 1 };
      this.showAddGuestModal = true;
    },
    addGuest(): void {
      const newTickets = Number(this.newGuest.tickets);
      const editingGuestTickets = this.editingIndex > -1 ? this.guests[this.editingIndex].tickets : 0;
      const totalTickets = this.totalGuests - editingGuestTickets + newTickets;

      if (totalTickets > 20) {
        alert('Sorry, the party can only fit 20 guests!');
        return;
      }

      if (this.editingIndex > -1) {
        this.guests.splice(this.editingIndex, 1, this.newGuest);
        this.editingIndex = -1;
      } else {
        this.guests.push(this.newGuest);
      }
      this.saveGuests();
      this.showAddGuestModal = false;
    },
    cancelAddGuest(): void {
      this.showAddGuestModal = false;
      this.editingIndex = -1;
    },
    async saveGuests(): Promise<void> {
      await new GuestRepository().save(this.guests);
    },
  },
});
</script>