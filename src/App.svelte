<script>
    import { afterUpdate, onMount, tick } from "svelte";
    import SearchResult from "./SearchResult.svelte";
    import Card from "./Card.svelte";

    let schemaVersion = "1";
    let imageURL = "";
    let queryString = "";
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
    const defaultCard = { front: cardBack, link: "", active: "front" };
    let selectedCard = defaultCard;
    let stuffheight;
    let historyHeight;
    let clickedCard = selectedCard;
    let activeIndex = 0;
    let highlightedSuggestion;

    const searchCards = async () => {
        const response = await fetch(
            "https://api.scryfall.com/cards/autocomplete?q=" + queryString,
        );
        const data = await response.json();

        suggestions = data.data;
        await tick();
        if (activeIndex == 0 && suggestionElements !== undefined) {
            if (suggestionElements.children.length > 0) {
                suggestionElements.children[0].style.background = "#ece8d9";
            }
        }
    };
    const handleClick = (event) => {
        fetchImages(event.detail.cardname);
    };

    const fetchImages = (cardName) => {
        fetch("https://api.scryfall.com/cards/named?exact=" + cardName)
            .then((response) => response.json())
            .then((data) => {
                console.log(data);
                let thisCard = {
                    front: "",
                    link: data.scryfall_uri,
                    active: "front",
                };
                try {
                    if ("card_faces" in data) {
                        thisCard.front = data.card_faces[0].image_uris.normal;
                        thisCard.back = data.card_faces[1].image_uris.normal;
                    } else {
                        thisCard.front = data.image_uris.normal;
                    }
                } catch (error) {
                    suggestions = [];
                    return;
                }

                cards.push(thisCard);

                // cards = cards;
                displayCards = cards.toReversed();
                suggestions = [];
                queryString = "";
                vis = "visible";

                clickedCard = thisCard;
                selectedCard = thisCard;
                localStorage.setItem(
                    "clickedCard",
                    JSON.stringify(clickedCard),
                );
                localStorage.setItem(
                    "selectedCard",
                    JSON.stringify(selectedCard),
                );
                localStorage.setItem("cards", JSON.stringify(cards));
                localStorage.setItem(
                    "displayCards",
                    JSON.stringify(displayCards),
                );
            })
            .catch((error) => console.error("Error:", error));
    };

    const flipCard = (event) => {
        selectedCard.active =
            selectedCard.active === "front" ? "back" : "front";
        displayCards = displayCards
    };

    const selectFirst = (event) => {
        if (event.key === "Enter" && suggestions.length > 0) {
            if (!highlightedSuggestion) {
                highlightedSuggestion = suggestions[0];
            }
            fetchImages(highlightedSuggestion);
            activeIndex = 0;
            highlightedSuggestion = null;
        } else if (event.key === "Escape" && suggestions.length > 0) {
            suggestions = [];
            activeIndex = 0;
            highlightedSuggestion = null;
        }

        if (event.key === "ArrowDown" && suggestions.length > 0) {
            suggestionElements.children[activeIndex].style.background = "white";
            if (activeIndex < suggestions.length - 1) {
                activeIndex++;
            }
            highlightedSuggestion = suggestions[activeIndex];
            suggestionElements.children[activeIndex].style.background =
                "#ece8d9";
        }
        if (event.key === "ArrowUp" && suggestions.length > 0) {
            suggestionElements.children[activeIndex].style.background = "white";
            if (activeIndex > 0) {
                activeIndex--;
            }
            highlightedSuggestion = suggestions[activeIndex];
            suggestionElements.children[activeIndex].style.background =
                "#ece8d9";
        }
    };

    onMount(async () => {
        try {
            localStorage.getItem("clickedCard")
                ? (clickedCard = JSON.parse(
                      localStorage.getItem("clickedCard"),
                  ))
                : null;
            localStorage.getItem("selectedCard")
                ? (selectedCard = JSON.parse(
                      localStorage.getItem("selectedCard"),
                  ))
                : null;
            localStorage.getItem("cards")
                ? (cards = JSON.parse(localStorage.getItem("cards")))
                : null;
            localStorage.getItem("displayCards")
                ? (displayCards = JSON.parse(
                      localStorage.getItem("displayCards"),
                  ))
                : null;
        } catch (error) {
            console.log(error);
        }
        if (localStorage.getItem("schemaVersion") !== schemaVersion) {
            clickClear();
            localStorage.setItem("schemaVersion", schemaVersion);
        }

        if (displayCards.length > 0) {
            vis = "visible";
        }
        padding.style.height = searchBox.offsetHeight + "px";
        textInputElement.focus();
    });

    afterUpdate(() => handleResize());

    const handleResize = (event) => {
        historyHeight = document.body.offsetHeight - stuffheight.offsetHeight;
        cardSpace = cardHeight / 4;
    };

    const refocus = (event) => {
        if (document.activeElement !== textInputElement) {
            textInputElement.focus();
        }
    };

    const clickClear = (event) => {
        cards = [];
        displayCards = [];
        selectedCard = defaultCard;
        vis = "hidden";
        console.log("click");

        localStorage.removeItem("clickedCard");
        localStorage.removeItem("selectedCard");
        localStorage.removeItem("cards");
        localStorage.removeItem("displayCards");
    };

    const cardMessage = (event) => {
        let event_type = event.detail.type;
        switch (event_type) {
            case "mouseenter":
                selectedCard = event.detail.data;
                break;
            case "mouseleave":
                selectedCard = clickedCard;
                break;
            case "click":
                clickedCard = event.detail.data;
                selectedCard = clickedCard;
                localStorage.setItem(
                    "clickedCard",
                    JSON.stringify(clickedCard),
                );
                localStorage.setItem(
                    "selectedCard",
                    JSON.stringify(selectedCard),
                );
            default:
        }
    };

    const openScryfall = (event) => {
        window.open(selectedCard.link);
    };
</script>

<head>
    <meta charset="utf-8" />
    <link
        rel="stylesheet"
        href="https://fonts.googleapis.com/css?family=Space Mono"
    />
</head>

<div class="searchbox" bind:this={searchBox}>
    <div class="searchboxinner">
        <input
            type="text"
            placeholder="search..."
            bind:this={textInputElement}
            bind:value={queryString}
            on:input={searchCards}
            on:keydown={selectFirst}
        />
        <div class="searchresultsbox" bind:this={suggestionElements}>
            {#each suggestions as cardname}
                <SearchResult on:message={handleClick} {cardname} />
            {/each}
        </div>
    </div>
</div>

<div
    class="clearbutton"
    style="position:absolute; bottom:10px; right:10px; z-index: 2; border:3px solid #494949; padding:5px; cursor: pointer; background-color: white"
    on:click={clickClear}
>
    clear
</div>

<div class="root" style="visibility:{vis}">
    <div bind:this={stuffheight}>
        <div class="searchboxpadding" bind:this={padding}></div>

        <div class="selectedcardbox">
            <div class="selectedcardelement">
                <div class="selectedcard">
                    <img
                        src={selectedCard[selectedCard.active]}
                        on:click={openScryfall}
                    />
                </div>
                <div class="flipButton" style="visibility:{ ('back' in selectedCard) ? 'visible' : 'hidden' }" on:click={flipCard}>🔁</div>
            </div>
        </div>
    </div>
    <div class="cardhistorycontainer">
        <div
            class="cardhistory"
            style="padding-top: {cardSpace / 4 + 35}px; height: {historyHeight -
                (cardSpace / 4 + 35)}px"
        >
            {#if displayCards.length > 0}
                {#each displayCards as card}
                    <Card {cardSpace} data={card} on:message={cardMessage} />
                {/each}
            {/if}
            <div
                class="cardspacer"
                style="height: {cardHeight}px; margin-top: -{cardSpace / 4}px"
            >
                <div class="card" bind:clientHeight={cardHeight}>
                    <img src={cardBack} alt="" />
                </div>
            </div>
        </div>
    </div>
</div>
<svelte:window on:resize={handleResize} on:keypress={refocus} />
