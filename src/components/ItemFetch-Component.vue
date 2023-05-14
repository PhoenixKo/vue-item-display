<template>
    <table v-if="!loading">
      <thead>
        <tr>
          <th>Item ID</th>
          <th></th>
          <th>Item Name</th>
          <th>Item Rarity</th>
          <th>API Link</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="itemId in itemIds" :key="itemId">
          <td>{{ itemId }}</td>
          <td>
            <img v-if="itemIconUrls[itemId]" :src="itemIconUrls[itemId]" alt="Item Icon" />     
            <span v-else>Item Icon Unavailable</span>
          </td>
          <td>{{ itemNames[itemId] }}</td>
          <td>{{ itemRarity[itemId] }}</td>
          <td>
            <a :href="getAPILink(itemId)" target="_blank">API Link</a>
          </td>
        </tr>
      </tbody>
    </table>
</template>

<style>
ul {
  list-style: none;
  padding: 0;
}

li {
  margin-bottom: 5px;
}

.loading {
  font-style: italic;
}

.error {
  color: red;
}
</style>

<script>
export default {
  data() {
    return {
      itemIds: [],
      itemIconUrls: {},
      itemNames: {},
      itemRarity: {},
      loading: false,
      error: null,
    };
  },
  mounted() {
    this.fetchItemIds();
  },
  methods: {
    fetchItemIds() {
      this.loading = true;
      fetch('https://api.guildwars2.com/v2/items?page=0&page_size=200')
        .then((response) => response.json())
        .then((data) => {
          this.itemIds = data.map((item) => item.id);
          this.fetchItemIconUrls();
          this.fetchItemNames();
          this.fetchItemRarity();
          this.loading = false;
        })
        .catch((error) => {
          console.error('Error:', error);
          this.error = error.message;
          this.loading = false;
        });
    },
    getItemIconUrl(itemId) {
      return new Promise((resolve, reject) => {
        const id = Number(itemId);

        if (isNaN(id) || !Number.isInteger(id)) {
          reject(new Error('Invalid item ID'));
          return;
        }

        fetch(`https://api.guildwars2.com/v2/items/${id}?lang=en&v=latest`)
          .then((response) => response.json())
          .then((data) => {
            const iconUrl = data.icon;
            resolve(iconUrl);
          })
          .catch((error) => {
            console.error('Error', error);
            reject(error);
          });
      });
    },
    fetchItemIconUrls() {
      this.itemIds.forEach((id) => {
        this.getItemIconUrl(Number(id))
          .then((iconUrl) => {
            this.itemIconUrls[id] = iconUrl; 
          })
          .catch((error) => {
            console.error('Error:', error);
          });
      });
    },
    fetchItemNames() {
      this.itemIds.forEach((id) => {
        fetch(`http://api.guildwars2.com/v2/items/${id}?lang=en&v=latest`)
          .then((response) => response.json())
          .then((data) => {
            const itemName = data.name;
            this.itemNames[id] = itemName;
          })
          .catch((error) => {
            console.error('Error:' , error);
          });
      });
    },
    fetchItemRarity() {
      this.itemIds.forEach((id) => {
        fetch(`http://api.guildwars2.com/v2/items/${id}?lang=en&v=latest`)
          .then((response) => response.json())
          .then((data) => {
            const itemRarity = data.rarity;
            this.itemRarity[id] = itemRarity;
          })
          .catch((error) => {
            console.error('Error:' , error);
          });
      });
    },
    getAPILink(itemId) {
      return `https://api.guildwars2.com/v2/items/${itemId}`;
    }
  },
};
</script>
