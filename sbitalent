// ==UserScript==
// @name         北森iTalent全能学习助手 - 加速 & 自动点击
// @namespace    http://tampermonkey.net/
// @version      2.2
// @description  适配 cloud.italent.cn，自动加速视频（最高2倍），自动点击"继续学习"、"确定"等弹窗，支持后台播放，防止倍速报错。
// @author       SB人事
// @match        http*://cloud.italent.cn/*
// @grant        none
// @license      MIT
// @run-at       document-idle
// ==/UserScript==

(function () {
    'use strict';

    const CONFIG = {
        VIDEO_SPEED: 2,                   
        AUTO_CLICK_DELAY: 500,
        DEBUG: false,
        TARGET_TEXTS: [
            '继续学习', '进入考试', '开始学习', '确定', '确认', '知道了', '我知道了', '好的', '是的'
        ],
        MODAL_BTN_SELECTOR: '.phoenix-modal--show .modal-btn'
    };

    let currentSpeed = CONFIG.VIDEO_SPEED;

    /**
     * 1. 设置视频倍速（强制限制在 0.5 ~ 2.0）
     */
    function setVideoSpeed(speed) {
        const finalSpeed = Math.min(Math.max(speed, 0.5), 2); // 11
        if (finalSpeed !== speed && CONFIG.DEBUG) {
            console.warn(`⚠️ 视频倍速超出范围（${speed}x），已自动调整为 ${finalSpeed}x`);
        }

        const videos = document.querySelectorAll('video');
        videos.forEach(video => {
            if (!video._gig_speed_set) {
                const applySpeed = () => {
                    video.playbackRate = finalSpeed;
                    video._gig_speed_set = true;
                    if (CONFIG.DEBUG) console.log(`🎬 视频倍速已设为 ${finalSpeed}x`);
                };
                if (video.readyState > 0) {
                    applySpeed();
                } else {
                    video.addEventListener('loadedmetadata', applySpeed, { once: true });
                }
            }
        });
    }

    /**
     * 2. 专门点击 "继续学习" 模态框按钮
     */
    function clickContinueLearningModal() {
        const modalBtn = document.querySelector(CONFIG.MODAL_BTN_SELECTOR);
        if (modalBtn) {
            const modal = modalBtn.closest('.phoenix-modal--show');
            if (modal && modal.offsetParent !== null) {
                modalBtn.click();
                console.log('✅ 成功点击 "继续学习" 弹窗按钮');
                return true;
            }
        }
        return false;
    }

    /**
     * 3. 点击其他通用按钮 (确定, 进入考试等)
     */
    function clickGenericButtons() {
        const elements = document.querySelectorAll(`
            button,
            .phoenix-button__content,
            .opt,
            [class*="button"],
            [class*="btn"],
            .phoenix-modal__content span
        `);

        for (const el of elements) {
            const text = el.textContent.trim();
            if (CONFIG.TARGET_TEXTS.some(t => text.includes(t) || text === t)) {
                const clickable = el.closest('button, .phoenix-button, .opt, .modal-btn, [onclick], [role="button"], .phoenix-modal__content');
                if (clickable && clickable.offsetParent !== null && !clickable.disabled) {
                    clickable.click();
                    console.log(`✅ 成功点击按钮: "${text}"`);
                    return true;
                }
            }
        }
        return false;
    }

    /**
     * 4. 综合点击函数
     */
    function autoClick() {
        if (clickContinueLearningModal()) return;
        clickGenericButtons();
    }

    /**
     * 5. 创建控制面板
     */
    function createControlPanel() {
        const panel = document.createElement('div');
        panel.innerHTML = `
            <div id="talent-helper-panel" style="
                position: fixed; top: 10px; right: 10px; z-index: 999999; 
                background: #fff; border: 1px solid #ddd; border-radius: 8px; 
                padding: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.15);
                font-family: sans-serif; font-size: 14px; min-width: 220px;
            ">
                <h4 style="margin: 0 0 10px; font-size: 16px; color: #333;">学习助手 v2.2</h4>
                <div style="margin-bottom: 10px;">
                    <label style="display:inline-block;width:70px;">视频倍速:</label>
                    <input type="number" id="speed-input" value="${CONFIG.VIDEO_SPEED}" 
                           min="0.5" max="2" step="0.5" style="width:60px;"
                           title="北森系统仅支持最高2倍速，超过会报错">
                </div>
                <button id="manual-click" style="
                    width:100%; background:#007bff; color:white; border:none; 
                    padding:8px 0; border-radius:4px; cursor:pointer; margin-top:5px;
                ">手动点击弹窗</button>
            </div>
        `;
        document.body.appendChild(panel);

        document.getElementById('speed-input').addEventListener('change', (e) => {
            const speed = parseFloat(e.target.value);
            if (speed > 0) {
                currentSpeed = speed;
                setVideoSpeed(speed);
            }
        });

        document.getElementById('manual-click').addEventListener('click', autoClick);
    }

    /**
     * 6. 屏蔽 visibilitychange 事件，防止后台播放被暂停
     */
    const originalAddEventListener = EventTarget.prototype.addEventListener;
    EventTarget.prototype.addEventListener = function(type, listener, options) {
        if (type === 'visibilitychange' || type === 'webkitvisibilitychange') {
            if (CONFIG.DEBUG) console.log(`🚫 屏蔽了 visibilitychange 事件`);
            return;
        }
        return originalAddEventListener.call(this, type, listener, options);
    };

    /**
     * 7. 初始化
     */
    function init() {
        console.log('🚀 北森全能学习助手 v2.2 已启动');

        // ⚠️ 首次运行提示（仅一次）
        if (CONFIG.DEBUG === false && !localStorage.getItem('gig_speed_warning_shown_v22')) {
            alert('⚠️ 注意：北森系统仅支持最高 2 倍速播放！\n\n本脚本已自动限制在安全范围内，切勿尝试超过 2x，否则可能导致学习失败、视频重播或记录异常！');
            localStorage.setItem('gig_speed_warning_shown_v22', 'true');
        }

        // 立即设置视频速度
        setVideoSpeed(currentSpeed);

        // 创建控制面板
        createControlPanel();

        // 使用 MutationObserver 监听 DOM 变化
        const observer = new MutationObserver(() => {
            setTimeout(autoClick, CONFIG.AUTO_CLICK_DELAY);
        });

        observer.observe(document.body, {
            childList: true,
            subtree: true
        });

        // 备用：定期检查按钮
        setInterval(autoClick, 2000);
    }

    // ========== 启动脚本 ==========
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', init);
    } else {
        init();
    }

})();
