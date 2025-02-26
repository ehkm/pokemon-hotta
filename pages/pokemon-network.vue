<template>
  <div>
    <h1>ポケモングラフ</h1>
    <svg ref="svgRef"></svg>
    <GamifyButton @click="goBack">もどる</GamifyButton>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import * as d3 from 'd3';

const route = useRoute();
const router = useRouter();
const svgRef = ref(null);
const pokemonNames = ref(route.query.names ? route.query.names.split(',') : []);

// ポケモンとタイプの関係を取得
const findPokemon = async (name) => {
  const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${name}`);
  if (!response.ok) {
    throw new Error(`Failed to fetch data for ${name}`);
  }
  return await response.json();
};

const fetchPokemonData = async (name) => {
  const data = await findPokemon(name);
  return {
    name: data.name,
    types: data.types.map(t => t.type.name),
    sprite: data.sprites.front_default, // 画像URLを追加
  };
};

// ネットワークグラフを描画
const drawGraph = async () => {
  const allData = await Promise.all(pokemonNames.value.map(fetchPokemonData));
  const nodes = [];
  const links = [];

  allData.forEach(data => {
    nodes.push({ id: data.name, type: 'pokemon', sprite: data.sprite });
    data.types.forEach(type => {
      if (!nodes.find(node => node.id === type)) {
        nodes.push({ id: type, type: 'type' });
      }
      links.push({ source: data.name, target: type });
    });
  });

  const width = 800;
  const height = 600;

  const svg = d3.select(svgRef.value)
    .attr('width', width)
    .attr('height', height)
    .call(d3.zoom().on('zoom', (event) => {
      svg.attr('transform', event.transform);
    }))
    .append('g');

  const simulation = d3.forceSimulation(nodes)
    .force('link', d3.forceLink(links).id(d => d.id).distance(100))
    .force('charge', d3.forceManyBody().strength(-300))
    .force('center', d3.forceCenter(width / 2, height / 2));

  const link = svg.selectAll('.link')
    .data(links)
    .enter()
    .append('line')
    .attr('stroke', '#aaa');

  const node = svg.selectAll('.node')
    .data(nodes)
    .enter()
    .append('g')
    .attr('class', 'node');

  node.append('circle')
    .attr('r', d => (d.type === 'pokemon' ? 10 : 7))
    .attr('fill', d => (d.type === 'pokemon' ? 'red' : 'blue'));

  node.filter(d => d.type === 'pokemon')
    .append('image')
    .attr('xlink:href', d => d.sprite)
    .attr('x', -60)  // 画像の位置を調整
    .attr('y', -60)  // 画像の位置を調整
    .attr('width', 120)  // 画像の幅を3倍に
    .attr('height', 120);  // 画像の高さを3倍に

  const text = svg.selectAll('.text')
    .data(nodes)
    .enter()
    .append('text')
    .attr('dx', 12)
    .attr('dy', 4)
    .text(d => d.id);

  simulation.on('tick', () => {
    link.attr('x1', d => d.source.x)
        .attr('y1', d => d.source.y)
        .attr('x2', d => d.target.x)
        .attr('y2', d => d.target.y);

    node.attr('transform', d => `translate(${d.x},${d.y})`);

    text.attr('x', d => d.x)
        .attr('y', d => d.y);
  });
};

const goBack = () => {
  router.go(-1);
};

onMounted(() => {
  console.log(pokemonNames.value);  // pokemonNamesが正しく取得されているか確認
  drawGraph();
});
</script>