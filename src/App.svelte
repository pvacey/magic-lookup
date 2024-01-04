<script>
  import {afterUpdate, onMount, tick} from 'svelte';
  import SearchResult from './SearchResult.svelte';

  let imageURL = '';
  let queryString = '';
  let suggestions = [];
  let suggestionElements;
  let vis = "hidden";
  let cards = [];
  let displayCards = [];
  // cards = [ "https://cards.scryfall.io/normal/front/a/e/aeece336-e5e8-4455-a297-c3739198d011.jpg?1674421574", "https://cards.scryfall.io/normal/front/4/c/4ccd0ada-92b2-48f3-b5ae-96346fc138b6.jpg?1673147890", "https://cards.scryfall.io/normal/front/8/4/8493131c-0a7b-4be6-a8a2-0b425f4f67fb.jpg?1689996248", "https://cards.scryfall.io/normal/front/4/f/4f893107-84b0-4e3f-b1f5-b04632f192c5.jpg?1699045094" ]
  let padding;
  let searchBox;
  let textInputElement;
  let cardSpace;
  let cardHeight;
  let cardBack = "https://backs.scryfall.io/large/2/2/222b7a3b-2321-4d4c-af19-19338b134971.jpg?1677416389";
  let selectedCard = "https://backs.scryfall.io/large/2/2/222b7a3b-2321-4d4c-af19-19338b134971.jpg?1677416389";
  let stuffheight;
  let historyHeight;
  let clickedCard = selectedCard;
  let activeIndex = 0;
  let highlightedSuggestion;

  const searchCards = async () => {
      const response = await fetch("https://api.scryfall.com/cards/autocomplete?q=" + queryString);
      const data = await response.json();

      suggestions = data.data;
      await tick();
      if (activeIndex == 0 && suggestionElements !== undefined) {
          if (suggestionElements.children.length > 0){
            suggestionElements.children[0].style.background='#ece8d9'
          }          
      }
  };
  const handleClick = (event) => {
      setImage(event.detail.cardname)
  };

  const setImage =  (cardName) => {
    fetch("https://api.scryfall.com/cards/named?exact=" + cardName)
          .then((response) => response.json())
          .then((data) => {
              if ("card_faces" in data){
                  console.log('yes, found it')
                  data = data.card_faces[0]
              }
              cards.push(data.image_uris.normal);
              // cards = cards;
              displayCards = cards.toReversed();
              suggestions = [];
              queryString = '';
              vis = 'visible';

              clickedCard = data.image_uris.normal;
              selectedCard = data.image_uris.normal;

              clickedCard = data.image_uris.normal;
              selectedCard = data.image_uris.normal;
              localStorage.setItem("clickedCard", JSON.stringify(clickedCard));
              localStorage.setItem("selectedCard", JSON.stringify(selectedCard));
              localStorage.setItem("cards", JSON.stringify(cards));
              localStorage.setItem("displayCards", JSON.stringify(displayCards));
          })
          .catch((error) => console.error('Error:', error));
  }

  
  const selectFirst = (event) => {
      // if (activeIndex == 0) {
      //     suggestionElements.children[0].style.background='#ece8d9'
      //     // suggestions = suggestions
      // }

      if (event.key === 'Enter' && suggestions.length > 0) {
          if (!highlightedSuggestion){
              highlightedSuggestion = suggestions[0]
          }
          setImage(highlightedSuggestion);
          activeIndex = 0;
          highlightedSuggestion = null;
      } else if (event.key === 'Escape' && suggestions.length > 0) {
          suggestions = [];
          activeIndex = 0;
          highlightedSuggestion = null;
      }

      if (event.key === 'ArrowDown' && suggestions.length > 0) {
          suggestionElements.children[activeIndex].style.background='white'
          if (activeIndex < suggestions.length -1 ){
              activeIndex++;
          }
          highlightedSuggestion = suggestions[activeIndex]
          suggestionElements.children[activeIndex].style.background='#ece8d9'
      }
      if (event.key === 'ArrowUp' && suggestions.length > 0) {
          suggestionElements.children[activeIndex].style.background='white'
          if (activeIndex > 0){
              activeIndex--;
          }
          highlightedSuggestion = suggestions[activeIndex]
          suggestionElements.children[activeIndex].style.background='#ece8d9'
      }
}
  
  onMount(async () => {
      try {
          localStorage.getItem("clickedCard") ? clickedCard =  JSON.parse(localStorage.getItem("clickedCard")) : null;
          localStorage.getItem("selectedCard") ? selectedCard =  JSON.parse(localStorage.getItem("selectedCard")) : null;
          localStorage.getItem("cards") ? cards =  JSON.parse(localStorage.getItem("cards")) : null;
          localStorage.getItem("displayCards") ? displayCards =  JSON.parse(localStorage.getItem("displayCards")) : null;            
      } catch (error) {
          console.log(error);
      }

      if (displayCards.length > 0) {
          vis = 'visible';
      }
      padding.style.height = (searchBox.offsetHeight) + 'px';
      textInputElement.focus();
  });

  afterUpdate(()=> handleResize())

  const handleResize = (event) => {
      historyHeight = document.body.offsetHeight  - stuffheight.offsetHeight
      cardSpace=(cardHeight/4);
  };

  const refocus = (event) => {
      if (document.activeElement !== textInputElement) {
          textInputElement.focus();
      }
  };
  
  const clickClear = (event) => {
      cards = [];
      displayCards = [];
      selectedCard = cardBack;
      vis = 'hidden';
      console.log('click');

      localStorage.removeItem("clickedCard");
      localStorage.removeItem("selectedCard");
      localStorage.removeItem("cards");
      localStorage.removeItem("displayCards");
  }

  const clickCard = (event) => {
      clickedCard = event.target.src;
      selectedCard = clickedCard
      localStorage.setItem("clickedCard", JSON.stringify(clickedCard));
      localStorage.setItem("selectedCard", JSON.stringify(selectedCard));
  };

  const mouseLeaveCard = (event) => {
      selectedCard = clickedCard
  };
  const mouseEnterCard = (event) => {
      selectedCard = event.target.src
  };

</script>

<head>
  <meta charset="utf-8">
  <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Space Mono">
</head>




<div class="searchbox" bind:this={searchBox}>
  <div class="searchboxinner">
      <input type="text" placeholder="search..." bind:this={textInputElement} bind:value={queryString} on:input={searchCards} on:keydown={selectFirst}>
      <div class="searchresultsbox" bind:this={suggestionElements}>
          {#each suggestions as cardname}
              <SearchResult on:message={handleClick} cardname={cardname}/>
          {/each}
      </div>
  </div>
</div>

<div class="clearbutton" style="position:absolute; bottom:10px; right:10px; z-index: 2; border:3px solid #494949; padding:5px; cursor: pointer; background-color: white" on:click={clickClear}>clear</div>

<div class="root" style="visibility:{vis}">
  <div bind:this={stuffheight}>
      
  <div class="searchboxpadding" bind:this={padding}></div>

  <div class="selectedcard">
      <img src={selectedCard}>
  </div>

  <div class="" style="text-align: center; padding:5px;">history</div>
  </div>
  <div class="cardhistorycontainer">
      <div class="cardhistory" style="padding-top: {(cardSpace/4)+35}px; height: {historyHeight - ((cardSpace/4)+35)}px">
          {#if displayCards.length > 0}
          {#each displayCards as card}
              <div class="cardspacer" style="height: {cardSpace}px; margin-top: -{cardSpace/4}px">
                  <div class="card" style="transform: rotate({(Math.random()*1.5) -1}deg) translateX({Math.random()*10}px);" >
                      <img src={card} alt="" on:mouseenter={mouseEnterCard} on:mouseleave={mouseLeaveCard} on:click={clickCard}>
                  </div>
              </div>
          {/each} 
          {/if}
          <div class="cardspacer" style="height: {cardHeight}px; margin-top: -{cardSpace/4}px">
              <div class="card" bind:clientHeight={cardHeight}>
                  <img src={cardBack} alt="">
              </div>
          </div>
      </div>
  </div>
  
</div>
<svelte:window on:resize={handleResize} on:keypress={refocus} />

<style>
  /* https://palettes.shecodes.io/palettes/1667 */
  input[type=text] {
      font-family: inherit;
      font-weight: inherit;
      font-size: large;
      width: 100%;
      padding: 12px 40px;
      box-sizing: border-box;
      border-left: none;
      border-right: none;
      border-top: 2px solid #494949;
      border-bottom: 2px solid #494949;
      /* border-radius: 5vmax; */
      outline: none;
      /* transition: 0.25s; */
      background-color: #494949;
      color: #fffdf6;
  }
  input[type=text]:focus {
      border-color: #494949;
      /* caret-color: transparent; */
  }input[type=text]:hover {
      border-color: #494949;
  }
  :global(body) {
      font-family: 'Space Mono', serif;
      font-weight: 300;
      font-size: large;
      margin: 0;
      background-color: #fffdf6;
  }
  .root{
      width: 100%;
      height: 100vh;
      box-sizing: border-box;
      color: #494949;
      display: flex;
      flex-direction: column;
      min-height:100%;
      
  }
  .searchbox{
      width: 100%;
      position: absolute;
      z-index: 1;
      filter: drop-shadow(0px 0px 5px #666);
  }
  .searchbox:hover{
  }
  .searchboxinner{
      width: 100%;
  }
  .searchresultsbox{
      background-color: #FFF;
      margin-right: inherit;
  }
  .searchboxpadding {
      /* padding-bottom: 20px; */
  }
  .cardhistorycontainer {
      display: flex;
      /* align-items: center; */
      justify-content: center;
  }
  .cardhistory{
      /* box-shadow: inset 0px 40px 20px -50px #666; */
      overflow-y: scroll;
      scrollbar-width: none;
      
      padding-right: 40px;
      padding-left: 40px;
  }
  .cardspacer{
      cursor: pointer;
  }
  .card {
      
      filter: drop-shadow(0px 0px 10px #666);
      position: relative;
  }
  .cardspacer:hover{
      cursor: pointer;
      filter: drop-shadow(0px 0px 5px white);
      transform: translateY(-10px) rotate(0.5deg);
      /* transform: translateY(-10px) perspective(400px) rotate3d(-12, 0 , 0, 5deg); */
  }
  .selectedcard {
      padding: 20px;
      filter: drop-shadow(0px 0px 10px #666);
  }
  img {
      display: block;
      margin-left: auto;
      margin-right: auto;
      max-width: 100%;
      height: auto;
      max-height: 45vh;
      border-radius: 4.75% 3.5%;
      
  }
</style>