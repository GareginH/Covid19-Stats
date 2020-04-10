<template>
  <div class="wrap">
    <div class="shadow">
      <h1>COVID-19 PANDEMIC</h1>
      <div class="flex" v-if="this.info.length > 2">
        <div>
          <h2>Most new deaths in</h2>
          <chart :dataset="prepareDataCases(4)" />
        </div>
        <div>
          <h2>Most new cases in</h2>
          <chart :dataset="prepareDataCases(2)" />
        </div>
        <div>
          <h2>Most critical cases in</h2>
          <chart :dataset="prepareDataCases(7)" />
        </div>
      </div>
      <div v-else style="margin-top:50px">
        <spinner
          size="massive"
          message="Loading data, please wait..."
        ></spinner>
      </div>
    </div>

    <div class="main-card" v-if="this.info.length > 2">
      <input
        class="search-field"
        type="text"
        placeholder="Filter by country"
        @change="searchCountry(country)"
        v-model="country"
      />
      <table class="table-center table-fit">
        <thead>
          <tr>
            <th>Country</th>
            <th>Infected</th>
            <th>New cases</th>
            <th>Total deaths</th>
            <th>New deaths</th>
            <th>Total recovered</th>
            <th>Active cases</th>
            <th>Critial cases</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, index) in this.searchInfo.length < 1
              ? this.info
              : this.searchInfo"
            :key="index"
          >
            <td>{{ item[0] }}</td>
            <td>{{ item[1] }}</td>
            <td class="dangerous">{{ item[2] }}</td>
            <td class="dangerous">{{ item[3] }}</td>
            <td class="dangerous">{{ item[4] }}</td>
            <td class="recovered">{{ item[5] }}</td>
            <td class="dangerous">{{ item[6] }}</td>
            <td class="dangerous">{{ item[7] }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import cheerio from "cheerio";
import chart from "./chart";
import Spinner from "vue-simple-spinner";
export default {
  name: "covidScrapper",
  data() {
    return {
      info: [],
      newDeaths: [],
      newCases: [],
      mostCritical: [],
      searchInfo: [],
      country: ""
    };
  },
  components: {
    chart,
    Spinner
  },
  created() {
    this.info = this.makeDataExportable();
  },
  mounted() {},
  methods: {
    searchCountry(country) {
      this.searchInfo = this.info.filter(
        el => el[0].toLowerCase().indexOf(country.toLowerCase()) >= 0
      );
    },
    prepareDataCases(type) {
      let list = [];
      let background = [
        `rgba(${Math.floor(Math.random() * 255 + 1)}, ${Math.floor(
          Math.random() * 255 + 1
        )}, ${Math.floor(Math.random() * 255 + 1)}, 1)`,
        `rgba(${Math.floor(Math.random() * 255 + 1)}, ${Math.floor(
          Math.random() * 255 + 1
        )}, ${Math.floor(Math.random() * 255 + 1)}, 1)`,
        `rgba(${Math.floor(Math.random() * 255 + 1)}, ${Math.floor(
          Math.random() * 255 + 1
        )}, ${Math.floor(Math.random() * 255 + 1)}, 1)`,
        `rgba(${Math.floor(Math.random() * 255 + 1)}, ${Math.floor(
          Math.random() * 255 + 1
        )}, ${Math.floor(Math.random() * 255 + 1)}, 1)`,
        `rgba(${Math.floor(Math.random() * 255 + 1)}, ${Math.floor(
          Math.random() * 255 + 1
        )}, ${Math.floor(Math.random() * 255 + 1)}, 1)`,
        `rgba(${Math.floor(Math.random() * 255 + 1)}, ${Math.floor(
          Math.random() * 255 + 1
        )}, ${Math.floor(Math.random() * 255 + 1)}, 1)`
      ];
      if (type === 4) {
        list = this.info.slice().sort((a, b) => b[4] - a[4]);
      } else if (type === 7) {
        list = this.info.slice().sort((a, b) => b[7] - a[7]);
      } else if (type === 2) {
        list = this.info.slice().sort((a, b) => b[2] - a[2]);
      }
      return {
        labels: [
          list[1][0],
          list[2][0],
          list[3][0],
          list[4][0],
          list[5][0],
          list[6][0]
        ],
        datasets: [
          {
            backgroundColor: background,
            hoverBorderColor: "rgba(0, 0, 0, 0.5)",
            data: [
              list[1][type],
              list[2][type],
              list[3][type],
              list[4][type],
              list[5][type],
              list[6][type]
            ]
          }
        ]
      };
    },
    async getSite(site) {
      return new Promise((resolve, reject) => {
        axios.get(site).then(data => {
          const $ = cheerio.load(data.data);
          let list = [];
          let currtext;
          $("table#main_table_countries_today>tbody>tr").each(
            (index, element) => {
              currtext = $(element)
                .find("td")
                .append("|")
                .text();
              if (currtext.indexOf("Total") < 0) {
                list.push(currtext);
              }
            }
          );
          resolve(list);
        });
      });
    },
    async makeDataExportable() {
      let data = await this.getSite(
        "https://www.worldometers.info/coronavirus/"
      );
      let list = data.map(e => {
        e = e.replace(/([,+])/g, "");
        return e.split("|");
      });
      this.info = list
        .filter(
          e =>
            e[0].indexOf("Asia") < 0 &&
            e[0].indexOf("Europe") < 0 &&
            e[0].indexOf("North America") < 0 &&
            e[0].indexOf("\n") < 0
        )
        .sort((a, b) => b[1] - a[1]);
    }
  }
};
</script>

<style scoped>
.main-card {
  border-radius: 10px;
  position: sticky;
  bottom: 0;
}
.search-field {
  border-radius: 5px;
  margin-top: 20px;
}
.table-center {
  margin: 0 auto;
  padding: 20px;
  padding-top: 0;
}
.table-fit {
  width: 100%;
}
.flex {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 90%;
}
.wrap {
  border-radius: 10px;
  width: 100%;
  margin: 0 auto;
}
.padding {
  padding-top: 40px;
}
.dangerous {
  color: red;
}
.recovered {
  color: green;
}
.shadow {
  -webkit-box-shadow: 0px 8px 5px 8px rgba(0, 0, 0, 0.4);
  -moz-box-shadow: 0px 8px 5px 8px rgba(0, 0, 0, 0.4);
  box-shadow: 0px 8px 5px 8px rgba(0, 0, 0, 0.4);
  margin-bottom: 10px;
  position: relative;
  top: 0;
  left: 0;
  z-index: 100;
  background: white;
  width: 100%;
  height: 780px;
}
h1 {
  margin: 0;
  padding-top: 40px;
}
table tbody,
table thead {
  display: block;
}
table tbody {
  overflow-y: auto;
  height: 500px;
}
th,
td {
  width: 110px;
}
</style>
