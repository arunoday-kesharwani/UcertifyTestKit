    <script>
    import {
        apiData,
        attempted,
        currentQues,
        unAttempted,
    } from './store/quesStore';
    import { userAnsObj } from './store/ansStore';
    import { afterUpdate } from 'svelte';
    import ReviewPage from './ReviewPage.svelte';
    let result = true;
    let review = false;
    export let quesExplMap = {};
    function onQuesClicked(event) {
        quesExplMap[event?.target?.dataset?.id] = true;
        result = false;
        review = true;
    }
    console.log($userAnsObj);

    function nextQuest(ques) {
        quesExplMap = {
        [$apiData[ques?.detail?.visibleQues]?.content_id]: true,
        };
    }

    function prevQuest(ques) {
        quesExplMap = {
        [$apiData[ques?.detail?.visibleQues]?.content_id]: true,
        };
    }
    export let correctAns = 0;
    export let inCorrectAns = 0;
    export let score = 0;
    afterUpdate(() => {
        Object.keys($userAnsObj).forEach((ques) => {
        if ($userAnsObj[ques].isCorrect == '1') {
            correctAns += 1;
        } else if ($userAnsObj[ques].isCorrect == '0') {
            inCorrectAns += 1;
        }
        score = ((correctAns / $apiData.length) * 100).toFixed(2);
        });
    });
    </script>

    <!-- svelte-ignore missing-declaration -->
    <header>
    <img id="logo" alt="uCertify Logo" src="Image/ucertifyLogo.png" />
    <h1 id="test-name">uCertify Result</h1>
    </header>
    {#if result}
    <div class="rsltDeclartion">
        <div class="data-item">
            <p>All Item</p>
            <p class="dataItem">{$apiData.length}</p>
        </div>
        <div class="data-item">
            <p>Attempted</p>
            <p class="dataItem">{$attempted}</p>
        </div>
        <div class="data-item">
            <p>UnAttempted</p>
            <p class="dataItem">{$unAttempted}</p>
        </div>
        <div class="data-item">
            <p>Correct Answer</p>
            <p class="dataItem">{correctAns}</p>
        </div>
        <div class="data-item">
            <p>InCorrect Answer</p>
            <p class="dataItem">{inCorrectAns}</p>
        </div>
    </div>

    <div><h2 class="totalResult">Total Result:{score}%</h2></div>
    <div class="outerContainer">
        <div class="container">
            <div class="column-left">Index No</div>
            <div class="column-center">Questions</div>
            <div class="column-right">Answers</div>
        </div>
        {#each $apiData as dataItem, i (dataItem)}
        <div class="container">
            <div class="column-left">{i + 1}</div>
            <!-- svelte-ignore a11y-missing-attribute -->
            <a
            class="column-center"
            on:click={onQuesClicked}
            data-id={dataItem.content_id}
            >{JSON.parse(dataItem.content_text).question}</a
            >

            <div class="column-right">
            {#each JSON.parse(dataItem.content_text).answers as ans, index (ans)}
                <h6
                class={ans.is_correct == '1'
                    ? 'correctCircleContainer'
                    : $userAnsObj.hasOwnProperty(i) &&
                    $userAnsObj[i].chosenAns == ans.answer
                    ? 'wrongCircleContainer'
                    : 'circleContainer'}
                >
                {index + 1}
            </h6>
            {/each}
            </div>
        </div>
        {/each}
    </div>
    {/if}
    {#if review}
    {#each $apiData as dataItem, i (dataItem)}
        {#if quesExplMap[dataItem.content_id]}
        <div class="outer"> 
        <div class="Explain-outer">
            <div class="question">
                {i+1}{'.    '}{JSON.parse(dataItem.content_text).question}
            </div>
            {#each JSON.parse(dataItem.content_text).answers as ans, index (ans)}
            <label
                for="ans{index}"
                id="option{index}"
                class={ans.is_correct == '1'
                ? 'correctLineContainer'
                : $userAnsObj.hasOwnProperty(i) &&
                    $userAnsObj[i].chosenAns == ans.answer
                ? 'wrongLineContainer'
                : 'LineContainer'}
            >
                <input
                type="radio"
                name="ans"
                id="ans{index}"
                class="selectAns"
                is_correct={ans.is_correct}
                value={ans.answer}
                checked={$userAnsObj.hasOwnProperty(i) &&
                    $userAnsObj[i].chosenAns == ans.answer}
                disabled
                />
                {@html ans.answer}
            </label>
            {/each}
            <div class="explanation"><strong>Explanation:</strong></div>
            <div class="explain-ans">
            {@html JSON.parse(dataItem.content_text).explanation}
            </div>
        </div>
        </div>
        <ReviewPage
            on:nextques={nextQuest}
            on:prevques={prevQuest}
            visibleQues={i}
        />
        {/if}
    {/each}
    {/if}

    <style>
    :global(seq:before) {
        content: attr(no);
    }
    #logo {
        position: relative;
        height: 100%;
        float: left;
    }
    .outer{
        overflow-y: hidden;
        height: 500px;
    }
    header {
        border: 2px solid rgb(0, 0, 0);
        width: 100%;
        top: 0;
        left: 0;
        height: 64px;
        box-shadow: 0 2px 6px rgba(0, 0, 0, 0.26);
    }

    #test-name {
        margin: 0;
        position: relative;
        color: rgb(197, 26, 26);
        height: 50px;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .rsltDeclartion {
        display: flex;
        flex-direction: row;
        justify-content: space-evenly;
        border: 2px solid #cf0056;
        margin: 40px;
    }

    .data-item {
        font-weight: bold;
        display: flex;
        flex-direction: column;
        text-align: center;
    }
    .explanation {
        border-top: 2px solid red;
        margin-top: 10px;
    }

    .totalResult {
        color: rgb(255, 0, 0);
        text-align: center;
    }

    .outerContainer {
        border: 2px solid rgb(0, 0, 0);
        margin-left: 100px;
        margin-right: 100px;
        /* padding: 5px; */
    }
    .column-left {
        float: left;
        width: 10%;
        /* border: 2px solid rgb(0, 0, 0); */
        margin: 2px;
        display: flex;
        flex-wrap: wrap;
        align-content: center;
        justify-content: space-evenly;
    }
    .column-right {
        float: right;
        width: 15%;
        /* border: 2px solid rgb(0, 0, 0); */
        margin: 2px;
        display: flex;
        flex-wrap: wrap;
        align-content: center;
        justify-content: space-evenly;
    }
    .column-center {
        display: inline-block;
        /* font-weight: bold; */
        width: 77.7%;
        text-align: center;
        border-left: 1.5px solid rgb(0, 0, 0);
        border-right: 1.5px solid rgb(0, 0, 0);
        margin: 2px;
    }
    .dataItem {
        display: inline-block;
    }
    .container {
        display: flex;
        flex-direction: row;
        justify-content: space-evenly;
        border: 1px solid black;
    }
    .circleContainer {
        border: 2px solid rgb(0, 0, 0);
        border-radius: 50%;
        padding: 12px;
    }
    .correctCircleContainer {
        border: 3px solid #56e056;
        border-radius: 50%;
        padding: 12px;
    }
    .wrongCircleContainer {
        border: 3px solid rgb(255, 0, 0);
        border-radius: 50%;
        padding: 12px;
    }
    .correctLineContainer {
        color: #56e056;
        margin-top: 2px;
    }
    .wrongLineContainer {
        color: rgb(255, 0, 0);
        margin-top: 2px;
    }

    .LineContainer,
    .explain-ans {
        margin-bottom: 2px;
        margin-top: 2px;
    }
    .Explain-outer {
        margin-top: 20px;
        padding: 5px;
        border: 2px solid rgb(255, 0, 0);
        
    }
    .question {
        padding-bottom: 5px;
    }

    </style>
