<script lang="ts">
  import { onMount } from 'svelte';

  let 억_임차보증금 = $state(2.1);
  let 만_월세 = $state(0);
  let 만_관리비 = $state(10);

  let 퍼_버팀목대출금리 = $state(1.8);
  let 억_버팀목대출한도 = $derived(Math.min(2, 억_임차보증금 * 0.8));
  let 만_버팀목_월이자 = $derived(
    ((억_버팀목대출한도 * (퍼_버팀목대출금리 / 100)) / 12) * 10000,
  );

  let 억_준비자금 = $derived(억_임차보증금 - 억_버팀목대출한도);
  let 퍼_신용대출금리 = $state(8);
  let 만_신용대출_월이자 = $derived(
    ((억_준비자금 * (퍼_신용대출금리 / 100)) / 12) * 10000,
  );

  let 만_월지출 = $derived(
    만_월세 + 만_관리비 + 만_신용대출_월이자 + 만_버팀목_월이자,
  );

  let name = $state('');

  interface Item {
    id: string;
    억_임차보증금: number;
    만_월세: number;
    만_관리비: number;
    name: string;
  }

  function getItems(): Item[] {
    return JSON.parse(localStorage.getItem('items') ?? '[]');
  }
  function setItems(items: Item[]) {
    localStorage.setItem('items', JSON.stringify(items));
  }
  function addItem(item: Item) {
    setItems([...getItems(), item]);
  }
  function removeItem(id: string) {
    setItems(getItems().filter((item) => item.id !== id));
  }
  let items = $state<Item[]>([]);

  onMount(() => {
    items = getItems();
    console.log(items);
  });
</script>

<main>
  <fieldset>
    <legend
      ><a href="https://nhuf.molit.go.kr/FP/FP05/FP0502/FP05020301.jsp"
        >청년전용 버팀목전세자금</a
      ></legend
    >

    <fieldset>
      <legend>연소득</legend>
      <label>
        <input
          type="radio"
          name="연소득"
          value={1.8}
          bind:group={퍼_버팀목대출금리}
        />
        2천만원 이하
      </label>
      <label>
        <input
          type="radio"
          name="연소득"
          value={2.1}
          bind:group={퍼_버팀목대출금리}
        />
        4천만원 이하
      </label>
      <label>
        <input
          type="radio"
          name="연소득"
          value={2.4}
          bind:group={퍼_버팀목대출금리}
        />
        9천만원 이하
      </label>
      <label>
        <input
          type="radio"
          name="연소득"
          value={2.7}
          bind:group={퍼_버팀목대출금리}
        />
        7.5천만원 이하
      </label>
    </fieldset>

    <fieldset>
      <label>
        전세|보증금
        <input type="number" bind:value={억_임차보증금} step={0.1} min={0} />억
      </label>
      <label>
        월세
        <input type="number" bind:value={만_월세} min={0} />만
      </label>
      <label>
        관리비
        <input type="number" bind:value={만_관리비} min={0} />만
      </label>
    </fieldset>
  </fieldset>

  <fieldset>
    <legend>신용대출</legend>
    <label>
      신용대출 금리
      <input type="number" bind:value={퍼_신용대출금리} min={0} step={0.1} />%
    </label>
  </fieldset>

  <div>
    <div>버팀목금리: {퍼_버팀목대출금리}%</div>
    <div>신용대출금리: {퍼_신용대출금리}%</div>
    <div>
      대출한도: {억_버팀목대출한도.toFixed(4)}억<span class="sub">
        &nbsp;× {퍼_버팀목대출금리}% = {Math.round(만_버팀목_월이자)}만</span
      >
    </div>
    <div>
      준비자금: {억_준비자금.toFixed(4)}억<span class="sub">
        &nbsp;× {퍼_신용대출금리}% = {Math.round(만_신용대출_월이자)}만</span
      >
    </div>
    <div>
      월 지출 비용: <span class="value">{Math.round(만_월지출)}만</span>
    </div>
    <div>
      2인 갹출: <span class="value">{Math.round(만_월지출 / 2)}만</span>
    </div>
  </div>
  <form
    onsubmit={(e) => {
      e.preventDefault();

      addItem({
        id: crypto.randomUUID(),
        억_임차보증금,
        만_월세,
        만_관리비,
        name,
      });

      items = getItems();
    }}
  >
    <label>
      이름
      <input bind:value={name} />
    </label>
    <button>저장</button>
  </form>

  <div class="items">
    {#each items as item}
      <div>
        {item.name}
        {#if item.만_월세}
          월세 {item.억_임차보증금}억/{item.만_월세}
        {:else}
          전세 {item.억_임차보증금}억
        {/if}
        <button
          onclick={() => {
            name = item.name;
            만_관리비 = item.만_관리비;
            만_월세 = item.만_월세;
            억_임차보증금 = item.억_임차보증금;
          }}>가져오기</button
        >
        <button
          onclick={() => {
            removeItem(item.id);
            items = getItems();
          }}>지우기</button
        >
      </div>
    {/each}
  </div>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    gap: 2rem;
  }
  fieldset {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
  }
  fieldset:has(fieldset) {
    flex-direction: column;
  }
  .value {
    font-weight: bold;
  }
  .sub {
    color: gray;
  }
</style>
