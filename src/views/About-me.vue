<template>
    <div class='wrap'>
        <div class='item'>
            <div class='ava'>
                <div>
                    <h2>Fullstack разработчик - Макшаков Сергей.</h2>
                    <p>dus-f@mail.ru</p>
                </div>

                <v-avatar size="130">
                    <img src="../assets/20181114-3.jpg" alt="Serj">
                </v-avatar>
            </div>
            <h3>Ссылки на мои работы:</h3>
            <hr/>
            <ul>
                <li>Лучшая разработка сайта: <a href="http://staraya-bashnya.ru" target="_blank">http://staraya-bashnya.ru</a></li>
                <li>SPA-продакшн на React js: <a href="http://pskvellift.ru/details/" target="_blank">http://pskvellift.ru/details/</a></li>
                <li>Парсинг Яндекса новостей на Node JS: <a href="https://still-chamber-91212.herokuapp.com/" target="_blank">https://still-chamber-91212.herokuapp.com/</a></li>
                <li>Онлайн чат на Node JS: <a href="https://frozen-mountain-01261.herokuapp.com/" target="_blank">https://frozen-mountain-01261.herokuapp.com/</a></li>
                <li>Виртуальный 3d-тур: <a href="http://staraya-bashnya.ru/virtual-tur.html" target="_blank">http://staraya-bashnya.ru/virtual-tur.html</a></li>
            </ul>
            <hr/>
            <p>Эти и другие мои работы на Github <a href="https://github.com/Serj-M/" target="_blank">https://github.com/Serj-M/</a></p>
        </div>
    </div>
</template>

<style scoped>
.wrap {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin: 70px 20px 40px;
    font-size: 21px;
    line-height: 40px;
}
.item {
    text-align: left;
}
.ava {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}
hr {
    margin: 20px 0;
}
</style>
