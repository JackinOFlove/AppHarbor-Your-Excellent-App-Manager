<template>
    <img class="background-image" src="@/assets/appharbor - admin background.svg">
    <div class="admin-page">
        <Header />
        <MainLayout />
    </div>
</template>

<script>
    import Header from "@/Views/AdminPage/Header.vue";
    import MainLayout from "@/Views/AdminPage/MainLayout.vue";

    export default {
        name: "AdminPage",
        components: {
            Header,
            MainLayout,
        },
    };
</script>

<style scoped>
    .admin-page {
        display: flex;
        flex-direction: column;
        height: 85vh;
        width: 100%;
        margin: 0;
        padding: 0;
    }
    .background-image {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
       
        background-size: cover;
        background-position: center;
        z-index: -1; /* ȷ������ͼƬ�ڵײ� */
    }

    header {
        flex-shrink: 0; 
    }

    main {
        flex-grow: 1; 
        overflow: hidden;
    }
</style>
