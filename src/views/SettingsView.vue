<template>
  <v-container>
    <v-card>
      <v-card-title>Settings</v-card-title>
      <v-card-text>
        <v-expansion-panels>
          <v-expansion-panel>
            <v-expansion-panel-header
              >Database Connection</v-expansion-panel-header
            >
            <v-expansion-panel-content>
              <v-form>
                <v-select
                  :items="databases"
                  label="Select a database"
                  v-model="selectedDatabase"
                  :disabled="connected"
                ></v-select>
                <template v-if="selectedDatabase === 'MySQL'">
                  <v-text-field
                    label="Hostname"
                    v-model="mysql.hostname"
                    :value="mysql.hostname"
                  ></v-text-field>
                  <v-text-field
                    label="Username"
                    v-model="mysql.username"
                    :value="mysql.username"
                  ></v-text-field>
                  <v-text-field
                    label="Password"
                    v-model="mysql.password"
                    type="password"
                    :value="mysql.password"
                  ></v-text-field>
                  <v-text-field
                    label="Database"
                    v-model="mysql.database"
                    :value="mysql.database"
                  ></v-text-field>
                </template>
                <template v-if="selectedDatabase === 'PostgreSQL'">
                  <v-text-field
                    label="Hostname"
                    v-model="postgres.hostname"
                    :value="postgres.hostname"
                  ></v-text-field>
                  <v-text-field
                    label="Username"
                    v-model="postgres.username"
                    :value="postgres.username"
                  ></v-text-field>
                  <v-text-field
                    label="Password"
                    v-model="postgres.password"
                    :value="postgres.password"
                    type="password"
                  ></v-text-field>
                  <v-text-field
                    label="Database"
                    v-model="postgres.database"
                    :value="postgres.database"
                  ></v-text-field>
                </template>
                <template v-if="selectedDatabase === 'MongoDB'">
                  <v-text-field
                    label="Connection string"
                    v-model="mongodb.connectionString"
                    :value="mongodb.connectionString"
                    :disabled="connected"
                  ></v-text-field>
                </template>
                <template v-if="selectedDatabase === 'Oracle'">
                  <v-text-field
                    label="Hostname"
                    v-model="oracle.hostname"
                    :value="oracle.hostname"
                  ></v-text-field>
                  <v-text-field
                    label="Username"
                    v-model="oracle.username"
                    :value="oracle.username"
                  ></v-text-field>
                  <v-text-field
                    label="Password"
                    v-model="oracle.password"
                    :value="oracle.password"
                    type="password"
                  ></v-text-field>
                  <v-text-field
                    label="Service name"
                    v-model="oracle.serviceName"
                    :value="oracle.serviceName"
                  ></v-text-field>
                </template>
                <v-btn
                  v-if="!connected"
                  color="primary"
                  @click="connect(selectedDatabase)"
                >
                  Connect
                </v-btn>
                <v-btn
                  v-if="connected"
                  :disabled="connected"
                  color="primary"
                  @click="connect(selectedDatabase)"
                >
                  Connected
                </v-btn>
              </v-form>
            </v-expansion-panel-content>
          </v-expansion-panel>
          <v-expansion-panel>
            <v-expansion-panel-header
              >Moodle Connection</v-expansion-panel-header
            >
            <v-expansion-panel-content>
              <v-form>
                <v-text-field
                  v-model="moodleUrl"
                  label="Moodle URL"
                ></v-text-field>
                <v-text-field
                  v-model="moodleToken"
                  label="Moodle Token"
                ></v-text-field>
                <v-btn
                  :disabled="moodleConnected"
                  color="primary"
                  @click="connectToMoodle"
                  >Connect</v-btn
                >
              </v-form>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
// import { mapState } from 'vuex';
import axios from "axios";

export default {
  data() {
    return {
      databases: ["MySQL", "PostgreSQL", "MongoDB", "Oracle"],
      selectedDatabase: "",
      moodleUrl: "",
      moodleToken: "",
      mysql: {
        hostname: "",
        username: "",
        password: "",
        database: "",
      },
      postgres: {
        hostname: "",
        username: "",
        password: "",
        database: "",
      },
      mongodb: {
        connectionString: "",
      },
      oracle: {
        hostname: "",
        username: "",
        password: "",
        serviceName: "",
      },
      connected: false,
      moodleConnected: false,
    };
  },
  computed: {
    // ...mapState({
    //   connected: state => state.database.connected,
    // }),
  },
  async created() {
    await axios.get("http://localhost:8082/api/connection").then((response) => {
      console.log(response.data);
      if (response.data.type == "mysql") {
        this.selectedDatabase = "MySQL";
        this.mysql.database = response.data.connection;
        this.mysql.hostname = response.data.hostname;
        this.mysql.username = response.data.username;
        this.mysql.password = response.data.password;
        this.connected = true;
      } else if (response.data.type == "mongodb") {
        this.selectedDatabase = "MongoDB";
        this.mongodb.connectionString = response.data.connection;
        this.connected = true;
      } else if (response.data.type == "postgress") {
        this.selectedDatabase = "PostgreSQL";
        this.postgres.database = response.data.connection;
        this.postgres.hostname = response.data.hostname;
        this.postgres.username = response.data.username;
        this.postgres.password = response.data.password;
        this.connected = true;
      } else if (response.data.type == "oracle") {
        this.selectedDatabase = "Oracle";
        this.oracle.serviceName = response.data.connection;
        this.oracle.hostname = response.data.hostname;
        this.oracle.username = response.data.username;
        this.oracle.password = response.data.password;
        this.connected = true;
      }
    });
  },
  methods: {
    async connect(database) {
      let db;
      database == "MySQL"
        ? (db = this.mysql)
        : database == "PostgreSQL"
        ? (db = this.postgres)
        : database == "MongoDB"
        ? (db = this.mongodb)
        : database == "Oracle"
        ? (db = this.oracle)
        : (db = null);
      console.log(db);
      try {
        console.log("Connecting to database...");
        await axios.post("http://localhost:8082/api/db-connect", {
          database: db.connectionString,
          type: database.toLowerCase(),
        });
        this.$store.dispatch("database/connect", this.selectedDatabase);
        this.connected = true;
      } catch (error) {
        console.error(error);
      }
    },
    connectToMoodle() {
      console.log(
        `Connecting to Moodle at ${this.moodleUrl} with token ${this.moodleToken} and course ${this.moodleCourse}`
      );
      // Send a request to the API to connect to Moodle using the provided URL, token, and course
    },
  },
};
</script>
