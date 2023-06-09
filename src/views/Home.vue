<template>
  <Page title="Board">
    <template #options>
      <BoardActions @input="changeHandler" />
    </template>
    <template #content>
      <div class="board">
        <BoardColumn
          v-for="column in boardsColumns"
          ref="list"
          :id="`column-${column.id}`"
          :key="column.id"
          :column="column"
          :hasCards="getColumnCards(column.id).length > 0"
          @load-more="fetchMoreCards"
        >
          <BoardCard
            :columnLabel="column"
            v-for="card in getColumnCards(column.id)"
            :key="card.id"
            :card="card"
            :column="column"
            @move-card="moveCard"
            :lists="[].slice.call(lists)"
          />
        </BoardColumn>
      </div>
      <router-view />
    </template>
  </Page>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, computed } from "vue";
import { State } from "@/store";
import { useStore } from "vuex";

import { Card, Column, MoveCardPayload } from "@/types/generics";
import Page from "@/components/generics/Page.vue";
import BoardColumn from "@/components/Board/BoardColumn.vue";
import BoardCard from "@/components/Board/BoardCard.vue";
import BoardActions from "@/components/Board/BoardActions.vue";

export default defineComponent({
  name: "HomeView",
  components: {
    Page,
    BoardCard,
    BoardColumn,
    BoardActions,
  },
  setup() {
    const store = useStore<State>();

    const list = ref<Card[] | null>(null);
    let lists = ref<Element[]>([]);

    const cards = computed<Card[]>(() => {
      return store.state.board.cards;
    });

    const boardColumns = computed<Column[]>(() => {
      return store.state.board.boardsColumns;
    });

    const getColumnCards = (columnId: number) => {
      return cards.value.filter((card) => card.column_id === columnId);
    };

    const moveCard = async (data: MoveCardPayload) => {
      try {
        const cardIdx = cards.value.findIndex(
          (card) => card.id === data.card.id
        );
        if (cardIdx >= 0) {
          cards.value[cardIdx].column_id = data.to_column_id;
        }
        await store.dispatch("board/updateCard", data);
      } catch (e) {
        console.error(e);
      }
    };

    const fetchColumns = async () => {
      await store.dispatch("board/fetchBoardColumns");
    };

    const fetchMoreCards = (columnId: number) => {
      console.log("Fetch more cards", columnId);
    };

    const fetchCards = async () => {
      await store.dispatch("board/fetchCards");
    };

    let debounceTimer: any = null;
    const debounceSearch = async (e: any) => {
      let searchQuery = e.target.value;
      clearTimeout(debounceTimer);

      if (searchQuery.length >= 3) {
        debounceTimer = setTimeout(() => {
          setTimeout(() => {
            store.state.board.cards = cards.value.filter((card) => {
              return card.title
                .toLowerCase()
                .includes(searchQuery.toLowerCase());
            });
          }, 500);
        }, 300);
      } else {
        await store.dispatch("board/fetchCards");
      }
    };

    const changeHandler = (e: any) => {
      debounceSearch(e);
    };

    onMounted(async () => {
      await Promise.all([fetchColumns(), fetchCards()]);
      if (list.value) {
        lists.value = [].slice.call(list.value);
      }
    });

    return {
      list,
      lists,
      boardsColumns: boardColumns,
      loadedCards: cards,
      moveCard,
      getColumnCards,
      changeHandler,
      fetchMoreCards,
    };
  },
});
</script>
<style scoped lang="scss">
@import "@/assets/styles/constants.scss";

.board {
  display: flex;
  flex-wrap: nowrap;
  padding: 16px 20px;
  height: 100%;
  width: 100%;
  overflow-x: auto;
  @include scrollbar();

  .column {
    width: 360px;
    flex-shrink: 0;
  }

  & > * + * {
    margin-left: 8px;
  }
}
</style>
