<template>
  <div class="wrap">
    <div class="shadow">
      <h1>COVID-19 PANDEMIC</h1>
      <div class="flex" v-if="this.prepareData() != null">
        <div>
          <h2>Most new deaths in</h2>
          <chart :dataset="this.prepareDataCases(4)" />
        </div>
        <div>
          <h2>Most new cases in</h2>
          <chart :dataset="this.prepareDataCases(2)" />
        </div>
        <div>
          <h2>Most critical cases in</h2>
          <chart :dataset="this.prepareDataCases(7)" />
        </div>
      </div>
    </div>

    <div class="main-card">
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
    chart
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
      let background = [];
      if (type === 4) {
        list = this.info.slice().sort((a, b) => b[4] - a[4]);
        background = [
          "rgba(102, 0, 204, 1)",
          "rgba(40, 232, 135, 1)",
          "rgba(255, 106, 186, 1)",
          "rgba(175, 92, 192, 1)",
          "rgba(223, 12, 20, 1)",
          "rgba(23, 212, 155, 1)"
        ];
      } else if (type === 7) {
        list = this.info.slice().sort((a, b) => b[7] - a[7]);
        background = [
          "rgba(255, 99, 132, 1)",
          "rgba(54, 162, 235, 1)",
          "rgba(255, 126, 26, 1)",
          "rgba(215, 92, 192, 1)",
          "rgba(153, 102, 255, 1)",
          "rgba(123, 112, 155, 1)"
        ];
      } else if (type === 2) {
        list = this.info.slice().sort((a, b) => b[2] - a[2]);
        background = [
          "rgba(155, 99, 232, 1)",
          "rgba(54, 162, 0, 1)",
          "rgba(50, 206, 86, 1)",
          "rgba(75, 90, 192, 1)",
          "rgba(123, 102, 155, 1)",
          "rgba(123, 0, 155, 1)"
        ];
      }
      try {
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
      } catch (e) {
        console.log(e);
      }
    },
    async getSite(site) {
      return new Promise((resolve, reject) => {
        axios.get(site).then(data => {
          const $ = cheerio.load(data.data);
          let list = [];
          $("table#main_table_countries_today>tbody>tr").each(
            (index, element) => {
              list.push(
                $(element)
                  .find("td")
                  .append("|")
                  .text()
              );
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
      list = list.sort((a, b) => b[1] - a[1]);
      this.info = list;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
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
  height: 800px;
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
