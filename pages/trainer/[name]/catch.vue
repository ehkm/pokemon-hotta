<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useRuntimeConfig } from '#app';

const route = useRoute();
const router = useRouter();
const config = useRuntimeConfig();

const allPokemons = ref([]);
const randomPokemons = ref([]);
const selectedPokemon = ref(null);
const failDialog = ref(false);

// すべてのポケモンを取得
const fetchAllPokemons = async () => {
  const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=10000');
  const data = await response.json();
  allPokemons.value = data.results;
  shufflePokemons();
};

// 10匹ランダムに選ぶ
const shufflePokemons = () => {
  if (allPokemons.value.length === 0) return;
  const shuffled = [...allPokemons.value].sort(() => Math.random() - 0.5);
  randomPokemons.value = shuffled.slice(0, 10);
};

// ポケモンの詳細を取得
const fetchPokemonDetails = async (pokemon) => {
  const response = await fetch(pokemon.url);
  const data = await response.json();
  selectedPokemon.value = {
    name: data.name,
    types: data.types.map(t => t.type.name), // タイプを取得
  };
};

// 捕まえる処理
const onCatch = async () => {
  if (!selectedPokemon.value) return;

  const isSuccess = Math.random() > 0.5;
  if (!isSuccess) {
    failDialog.value = true;
    return;
  }

  const response = await $fetch(`/api/trainer/${route.params.name}/pokemon`, {
    baseURL: config.public.backendOrigin,
    method: "POST",
    body: { name: selectedPokemon.value.name },
  }).catch((e) => e);

  if (response instanceof Error) return;
  router.push(`/trainer/${route.params.name}`);
};

// 初期データ取得
onMounted(fetchAllPokemons);
</script>

<template>
  <div>
    <h1>ポケモンをつかまえる</h1>
    <p>{{ allPokemons.length }} しゅるいのポケモン</p>
    <GamifyButton @click="shufflePokemons">またさがす</GamifyButton>

    <GamifyList>
      <GamifyItem v-for="pokemon in randomPokemons" :key="pokemon.url">
        <span class="pokemon-name">{{ pokemon.name }}</span>
        <GamifyButton @click="fetchPokemonDetails(pokemon)">つかまえる</GamifyButton>
      </GamifyItem>
    </GamifyList>

    <!-- 確認ダイアログ -->
    <GamifyDialog
      v-if="selectedPokemon"
      id="confirm-catch"
      title="かくにん"
      :description="`ほう！　${selectedPokemon.name} (${selectedPokemon.types.join(', ')})　にするんじゃな？`"
      @close="selectedPokemon = null"
    >
      <GamifyList :border="false" direction="horizon">
        <GamifyItem>
          <GamifyButton @click="selectedPokemon = null">いいえ</GamifyButton>
        </GamifyItem>
        <GamifyItem>
          <GamifyButton @click="onCatch">はい</GamifyButton>
        </GamifyItem>
      </GamifyList>
    </GamifyDialog>

    <!-- 失敗ダイアログ -->
    <GamifyDialog
      v-if="failDialog"
      id="fail-catch"
      title="しっぱい"
      description="にげられた！"
      @close="failDialog = false"
    >
      <GamifyButton @click="failDialog = false">とじる</GamifyButton>
    </GamifyDialog>
  </div>
</template>