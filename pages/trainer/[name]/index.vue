<script setup>
const route = useRoute();
const router = useRouter();
const config = useRuntimeConfig();
const { data: trainer, refresh } = await useFetch(
  () => `/api/trainer/${route.params.name}`,
  {
    default: () => [],
    server: false,
    baseUrl: config.public.backendOrigin,
  },
);
const onDelete = async () => {
  const response = await $fetch(`/api/trainer/${route.params.name}`, {
    baseURL: config.public.backendOrigin,
    method: "DELETE",
  }).catch((e) => e);
  if (response instanceof Error) return;
  router.push("/");
};
const nickname = ref("");
const onNickname = async (pokemon) => {
  const newTrainer = trainer.value;
  const index = newTrainer.pokemons.findIndex(({ id }) => id === pokemon.id);
  newTrainer.pokemons[index].nickname = trimAvoidCharacters(nickname.value);
  nickname.value = "";
  const response = await $fetch(`/api/trainer/${route.params.name}`, {
    baseURL: config.public.backendOrigin,
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(newTrainer),
  }).catch((e) => e);
  if (response instanceof Error) return;
  await refresh();
  onCloseNickname();
};
const onRelease = async (pokemonId) => {
  const response = await fetch(
    `/api/trainer/${route.params.name}/pokemon/${pokemonId}`,
    {
      baseURL: config.public.backendOrigin,
      method: "DELETE",
    },
  ).catch((e) => e);
  if (response instanceof Error) return;
  await refresh();
  onCloseRelease();
};
const viewPokemonGraph = () => {
  const pokemonNames = trainer.value.pokemons.map(pokemon => pokemon.name).join(',');
  router.push({ path: '/pokemon-network', query: { names: pokemonNames } });
};
const {
  dialog: deleteDialog,
  onOpen: onOpenDelete,
  onClose: onCloseDelete,
} = useDialog();
const {
  dialog: nicknameDialog,
  onOpen: onOpenNickname,
  onClose: onCloseNickname,
} = useDialog();
const {
  dialog: releaseDialog,
  onOpen: onOpenRelease,
  onClose: onCloseRelease,
} = useDialog();
</script>

<template>
  <div>
    <h1>トレーナー情報</h1>
    <div class="trainer-info">
      <img src="/avatar.png" />
      <span>{{ trainer.name }}</span>
    </div>
    <GamifyButton @click="onOpenDelete(true)"
      >マサラタウンにかえる</GamifyButton
    >
    <h2>てもちポケモン</h2>
    <div class="button-group">
      <CatchButton :to="`/trainer/${route.params.name}/catch`"
        >ポケモンをつかまえる</CatchButton
      >
      <GamifyButton @click="viewPokemonGraph">ポケモンを見る</GamifyButton>
    </div>
    <GamifyList>
      <GamifyItem v-for="pokemon in trainer.pokemons" :key="pokemon.id">
        <img :src="pokemon.sprites.front_default" />
        <span class="pokemon-name">{{ pokemon.nickname || pokemon.name }}</span>
        <GamifyButton @click="onOpenNickname(pokemon)"
          >ニックネームをつける</GamifyButton
        >
        <GamifyButton @click="onOpenRelease(pokemon)"
          >はかせにおくる</GamifyButton
        >
      </GamifyItem>
    </GamifyList>
  </div>
</template>

<style scoped>
.button-group {
  display: flex;
  gap: 1rem;
  margin-bottom: 1rem;
}
.trainer-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 1rem;
}
.trainer-info > img {
  width: 3rem;
  height: 3rem;
}
</style>