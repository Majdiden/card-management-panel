<template>
  <v-container>
    <v-card>
      <v-card-title>ID Card Management</v-card-title>
      <v-card-text>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          single-line
          hide-details
        ></v-text-field>
        <v-data-table
          :headers="headers"
          :items="students"
          :search="search"
          :items-per-page="5"
          class="elevation-1"
        >
          <template v-slot:[`item.actions`]="{ item }">
            <v-btn color="primary" @click="showIssueModal(item)"
              >Issue Card</v-btn
            >
          </template>
        </v-data-table>
        <v-dialog v-model="issueModal" max-width="400">
          <template v-slot:activator="{ on }">
            <v-btn v-on="on" color="primary" class="d-none"></v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">Issue ID Card</span>
            </v-card-title>
            <v-card-text>
              Scan your ID card to issue it.
              <br />
              <v-icon
                v-if="issuing"
                class="mr-2"
                color="primary"
                size="100"
                style="padding: 25%; overflow: hidden"
                >fas fa-spinner fa-spin</v-icon
              >
              <br />
              <v-icon
                v-if="!issuing && issued"
                color="success"
                size="150"
                style="padding: 25%; overflow: hidden"
                >fas fa-check</v-icon
              >
              <br />
              <v-icon
                v-if="!issuing && !issued"
                color="black"
                size="150"
                style="padding: 25%; overflow: hidden"
                >fas fa-sharp fa-solid fa-id-card</v-icon
              >
            </v-card-text>
            <v-card-actions>
              <v-btn text @click="closeIssueModal">Close</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      search: "",
      headers: [
        { text: "Name", value: "name" },
        { text: "ID", value: "std_id" },
        { text: "Actions", value: "actions", sortable: false },
      ],
      students: [],
      issueModal: false,
      issuing: false,
      issued: false,
      student: {},
    };
  },
  created() {
    this.fetchStudents();
  },
  methods: {
    showIssueModal(student) {
      this.student = student;
      this.issueModal = true;
      axios
        .get("http://localhost:8082/api/scan-to-issue")
        .then((response) => {
          console.log(response);
          if (response.data.message == "success") {
            this.issueCard(student);
          }
        })
        .catch((err) => console.log(err));
    },
    closeIssueModal() {
      this.issueModal = false;
    },
    async fetchStudents() {
      try {
        const response = await axios.get("http://localhost:8082/api/students", {
          params: {
            database: "mongodb",
            connectionString:
              "mongodb+srv://grad-proj:grad-proj-123@cluster1.1aizeuk.mongodb.net/test",
          },
        });
        this.students = response.data;
      } catch (error) {
        console.error(error);
      }
    },
    async issueCard(student) {
      this.issuing = true;
      console.log(student);
      try {
        await axios
          .post(`http://localhost:8082/api/${student.id}/issue-card`)
          .then((response) => {
            console.log(response);
            if (response.data.message == "ID Card Issued Successfully") {
              setTimeout(() => {
                this.issuing = false;
              }, 3000);
              this.issued = true;
              setTimeout(() => {
                this.issueModal = false;
              }, 6000);
            }
          });
      } catch (error) {
        console.error(error);
      }
    },
  },
  computed: {
    filteredStudents() {
      const search = this.search.toLowerCase();
      return this.students.filter((student) => {
        return (
          student.name.toLowerCase().includes(search) ||
          student.id.toLowerCase().includes(search)
        );
      });
    },
  },
};
</script>
