<template>
  <div class="servers">
    <div class="heading servers__heading">
      <div class="heading__data-box">
        <h2 class="heading__page-name">All Servers</h2>
        <div class="heading__servers-info">
          <t-icon class="heading__icon" icon="cloud-connection" />
          <span class="heading__servers-amount">{{ items.length }}</span>
          <span class="heading__servers-data-name">total servers</span>
        </div>
        <div class="heading__servers-info">
          <t-icon class="heading__icon" icon="arrow-right-square" />
          <span class="heading__servers-amount">{{ allocated }}</span>
          <span class="heading__servers-data-name">allocated</span>
        </div>
        <div class="heading__servers-info">
          <t-icon class="heading__icon" icon="box" />
          <span class="heading__servers-amount">{{ capacity }}</span>
          <span class="heading__servers-data-name">capacity</span>
        </div>
      </div>
      <div class="heading__filters-box">
        <t-select-list
          class="heading__filters-item"
          @checkedValue="setServersFilterOption"
          title="Servers"
          :defaultValue="TServersServersFilterOptions.ALL"
          :values="TServersServersFilterOptions"
        />
        <t-select-list
          class="heading__filters-item"
          @checkedValue="setStatusesFilterOption"
          title="Statuses"
          :defaultValue="TServersStatusesFilterOptions.ALL"
          :values="TServersStatusesFilterOptions"
        />
      </div>
    </div>
    <div class="servers__input-box">
      <t-input
        @input.self="setInputValue"
        type="secondary"
        placeholder="Search servers by name..."
        @clearInput="setInputValue"
      />
    </div>
    <t-servers-content-list :items="filteredItems" />
  </div>
</template>

<script>
import { computed, ref, toRefs } from "@vue/reactivity";
import TInput from "@/components/common/TInput/TInput.vue";
import TServersContentList from "./TServersContentList.vue";
import TIcon from "@/components/common/Icon/TIcon.vue";
import TSelectList from "@/components/common/SelectList/TSelectList.vue";
import {
  TServersServersFilterOptions,
  TServersStatusesFilterOptions,
} from "@/constants";
export default {
  components: { TInput, TServersContentList, TIcon, TSelectList },
  props: {
    items: Object,
  },
  setup(props) {
    const { items } = toRefs(props);
    const inputValue = ref("");
    const serversFilterOption = ref("All");
    const statusesFilterOption = ref("All");
    const setServersFilterOption = (data) => {
      serversFilterOption.value = String(data);
    };
    const setStatusesFilterOption = (data) => {
      statusesFilterOption.value = String(data);
    };

    const setInputValue = (data) => {
      inputValue.value = data;
    };

    const filteredItems = computed(() => {
      const arr =
        serversFilterOption.value !== TServersServersFilterOptions.ALL
          ? items?.value?.filter((elem) => {
              return (
                ((elem?.status || {})?.power === "on" ? "On" : "Off") ===
                  serversFilterOption.value &&
                elem?.metadata?.uid?.includes(inputValue.value)
              );
            })
          : items?.value;

      return statusesFilterOption.value !== TServersStatusesFilterOptions.ALL
        ? arr.filter((elem) => {
            return (
              (elem?.status?.ready ? "Ready" : "Failed") ===
                statusesFilterOption.value &&
              elem?.metadata?.uid?.includes(inputValue.value)
            );
          })
        : inputValue.value?.length === 0
        ? arr
        : arr.filter((elem) => {
            return elem?.metadata?.uid?.includes(inputValue.value);
          });
    });

    const allocated = computed(() => {
      return (items?.value || []).filter((elem) => elem?.status?.inUse).length;
    });

    return {
      setServersFilterOption,
      setStatusesFilterOption,
      setInputValue,
      filteredItems,
      TServersServersFilterOptions,
      TServersStatusesFilterOptions,
      allocated: allocated,
      capacity: computed(() => {
        if (items?.value?.length === 0) {
          return null;
        }

        return 100 - (allocated.value / items.value.length) * 100 + "%";
      }),
    };
  },
};
</script>

<style scoped>
.servers {
  @apply flex flex-col h-full;
  max-height: calc(100vh - 110px);
}
.servers__heading {
  margin-bottom: 29px;
}
.heading {
  @apply flex justify-between;
}
.heading__data-box {
  @apply flex items-center flex-wrap;
}
.heading__filters-box {
  @apply flex justify-end;
}
.heading__page-name {
  @apply text-xl text-naturals-N14 mr-8 font-medium;
}
.heading__servers-info {
  @apply flex items-center;
  margin-right: 20px;
}
.heading__icon {
  @apply fill-current text-primary-P3;
  width: 19px;
  height: 19px;
  margin-right: 6px;
}
.heading__servers-amount {
  @apply text-naturals-N14 font-medium;
  margin-right: 6px;
  font-size: 16px;
}
.heading__servers-data-name {
  @apply text-naturals-N9 font-normal;
  font-size: 16px;
}
.heading__filters-item:last-of-type {
  @apply ml-2;
}
.servers__input-box {
  @apply flex;
  margin-bottom: 31px;
}
</style>
