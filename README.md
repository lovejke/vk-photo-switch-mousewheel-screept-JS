# vk-photo-switch-mousewheel-screept-JS
vk photo switch (mousewheel) screept JS

==============================================================

// ==UserScript==
// @name         VK Photo MouseWheel Scroll (No Violations)
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Переключение фото в альбомах VK колесиком мыши, без нарушений производительности
// @author       Artem Sha. Asdweb.ru
// @match        https://vk.com/*
// @match        https://vkontakte.ru/*
// @match        https://vk.ru/*
// @grant        none
// ==/UserScript==


(function() {
    'use strict';

    function getPhotoViewerElements() {
        return {
            imageWrap: document.querySelector('.pv_image_wrap') ||
                         document.querySelector('[class*="pv_image"]'),
            nextBtn: document.getElementById('pv_nav_btn_right'),
            prevBtn: document.getElementById('pv_nav_btn_left')
        };
    }

    function handleWheel(event) {
        const { imageWrap, nextBtn, prevBtn } = getPhotoViewerElements();

        if (!imageWrap) return;

        const isInsideImageWrap = event.target.closest('.pv_image_wrap') ||
                       (imageWrap && imageWrap.contains(event.target));
        if (!isInsideImageWrap) return;

        const delta = event.deltaY;

        // Антидребезг: 200 мс между переключениями
        if (window.vkPhotoScrollLastScroll &&
            Date.now() - window.vkPhotoScrollLastScroll < 200) {
            return;
        }
        window.vkPhotoScrollLastScroll = Date.now();

        // Переключаем фото через API VK — без preventDefault()
        if (delta > 0) {
            console.log('✅ Switching to next photo via Photoview API');
            Photoview?.show(false, cur.pvIndex + 1, event);
        } else if (delta < 0) {
            console.log('✅ Switching to previous photo via Photoview API');
            Photoview?.show(false, cur.pvIndex - 1, event);
        }
    }

    // Функция управления обработчиком
    function manageHandler() {
        const elements = getPhotoViewerElements();

        if (elements.imageWrap && (elements.nextBtn || elements.prevBtn)) {
            if (!window.vkPhotoScrollHandlerAttached) {
                // ВАЖНО: добавляем { passive: true }
                document.addEventListener('wheel', handleWheel, { passive: true });
                window.vkPhotoScrollHandlerAttached = true;
                console.log('🟢 Wheel handler attached (passive)');
            }
        } else {
            if (window.vkPhotoScrollHandlerAttached) {
                document.removeEventListener('wheel', handleWheel);
                window.vkPhotoScrollHandlerAttached = false;
                console.log('🔴 Wheel handler detached');
            }
        }
    }

    // Запускаем сразу
    manageHandler();

    // Периодическая проверка каждые 250 мс
    setInterval(manageHandler, 250);

    // Наблюдатель за изменениями DOM
    const observer = new MutationObserver(manageHandler);
    observer.observe(document.body, {
        childList: true,
        subtree: true
    });

    // Проверка при фокусе окна
    window.addEventListener('focus', manageHandler);
})();
