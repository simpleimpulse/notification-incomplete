<template>
    <div id="app">
        <img alt="Notification Management Logo" src="./assets/logo.png" />
        <h1>Notification Management</h1>
        <section id="backend">
            <h3>Mock Backend Control Panel</h3>
            <MockBackend />
        </section>

        <section id="frontend">
            <h3>Frontend View</h3>

            <NotificationWidget />
            <OrganizationWidget />
        </section>
    </div>
</template>

<script>
    import OrganizationWidget from "./components/OrganizationWidget.vue";
    import NotificationWidget from "./components/NotificationWidget.vue";
    import MockBackend from "./components/MockBackend.vue";

    export default {
        name: "App",
        components: {
            OrganizationWidget,
            NotificationWidget,
            MockBackend
        },
        props: {
            title: String
        }
    };
</script>

<style>
    html,
    body {
        /* Reset */
        margin: 0;
        padding: 0;
    }
    h1 {
        margin: 10px 0 20px;
    }
    h3 {
        margin-bottom: 40px;
    }
    img {
        max-width: 100px;
    }
    ul {
        list-style: none;
        padding: 0;
        margin: 0;
    }
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 20px;
    }
    #backend,
    #frontend {
        min-height: 475px;
        width: 48%;
        margin: 0 1%;
        padding: 1%;
        box-sizing: border-box;
        display: inline-block;
        position: relative;
        vertical-align: top;
        box-shadow: 0px 0px 5px #ccc;
        border-radius: 5px;
    }
    header {
        position: absolute;
        height: 60px;
        width: 100%;
        top: 0;
    }
</style>
