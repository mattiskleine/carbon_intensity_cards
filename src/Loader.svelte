<script>
    export let styleSpinner
</script>

<div id="spinner" style={styleSpinner}>
    <div class="ring" id=ring1></div>
    <div class="ring" id=ring2></div>
    <div class="ring" id=ring3></div>
    <div id="text">loading database</div>
</div>

<style>
    #spinner {
        position: absolute;
        left: 47vw;
        top: 24vw;
        width: 6vw;
        height: 6vw;
        transition: transform .5s;
    }

    .ring {
        position: absolute;
        border-radius: 50%;
        border: 0.4vw solid;
        box-sizing: border-box;
        animation: spinning .6s linear infinite;
    }

    #ring1 {
        top: 20%;
        left: 20%;
        width: 60%;
        height: 60%;
        border-color: #D3DEDC;
        border-left-color: transparent;
    }

    #ring2 {
        top: 10%;
        left: 10%;
        width: 80%;
        height: 80%;
        border-color: #7C99AC;
        border-right-color: transparent;
    }

    #ring3 {
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        border-color: #92A9BD;
        border-left-color: transparent;
    }

    #text {
        position: absolute;
        width: 100%;
        text-align: center;
        font-size: 1.1vw;
        bottom: -4vw;
    }

    @keyframes spinning {
        0%   {transform: rotate(0)}
        100% {transform: rotate(360deg)}
}
</style>
