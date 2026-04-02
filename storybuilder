// ==UserScript==
// @name         크랙 스토리 빌더 수정도구(제작자용)
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  사이트 업데이트 대응: UI가 삭제되어도 자동으로 다시 생성하는 불사신 로직 적용
// @author       CDD
// @match        https://crack.wrtn.ai/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    const TOOL_ID = 'wrtn-master-tool-v8';

    // 사람의 마우스 클릭 흉내내기 함수
    const simulateRealClick = (element) => {
        if (!element) return;
        ['mouseover', 'mousedown', 'mouseup', 'click'].forEach(eventType => {
            element.dispatchEvent(new MouseEvent(eventType, { bubbles: true, cancelable: true, buttons: 1 }));
        });
    };

    // UI를 화면에 그리는 함수 (삭제되면 다시 그림)
    const renderUI = () => {
        // 이미 화면에 도구창이 있으면 중복으로 그리지 않음
        if (document.getElementById(TOOL_ID)) return;

        // 스타일 태그 주입 (없을 때만)
        if (!document.getElementById('wrtn-style-v8')) {
            const style = document.createElement('style');
            style.id = 'wrtn-style-v8';
            style.innerHTML = `
                #${TOOL_ID} { position: fixed; top: 100px; right: 20px; z-index: 999999; background: #1e272e; padding: 15px; border-radius: 12px; display: flex; flex-direction: column; gap: 10px; box-shadow: 0 10px 30px rgba(0,0,0,0.6); border: 1px solid #485460; width: 230px; }
                .m-btn { padding: 12px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; color: white; transition: 0.2s; font-size: 13px; }
                .m-del { background: #ff3f34; }
                .m-sort { background: #0fb8ce; }
                .m-toggle { background: #05c46b; }
                .m-fill { background: #f39c12; }
                .m-btn:hover { filter: brightness(1.2); transform: translateY(-1px); }
                .m-btn:disabled { background: #808e9b; cursor: wait; }
            `;
            document.head.appendChild(style);
        }

        const container = document.createElement('div');
        container.id = TOOL_ID;

        // 1. 전체 접기/펼치기
        const toggleBtn = document.createElement('button');
        toggleBtn.innerText = '↕️ 전체 창 축소/확대';
        toggleBtn.className = 'm-btn m-toggle';
        toggleBtn.onclick = () => {
            const toggleButtons = Array.from(document.querySelectorAll('button')).filter(btn => {
                const svg = btn.querySelector('svg');
                return svg && (svg.innerHTML.includes('m12 7.37 6.36') || (svg.innerHTML.includes('1.14L12') && !svg.innerHTML.includes('M15.44 4.34')));
            });
            if (toggleButtons.length === 0) { alert('축소/확대 버튼을 찾지 못했습니다.'); return; }
            toggleButtons.forEach(btn => simulateRealClick(btn));
        };

        // 2. 이미지 전체 삭제
        const deleteBtn = document.createElement('button');
        deleteBtn.innerText = '❌이미지 전체 삭제';
        deleteBtn.className = 'm-btn m-del';
        deleteBtn.onclick = async () => {
            const getTargets = () => Array.from(document.querySelectorAll('button')).filter(btn => {
                const svg = btn.querySelector('svg');
                return svg && (svg.innerHTML.includes('M15.44 4.34H8.56') || svg.innerHTML.includes('M2.6 5.43v1.6'));
            });

            let targets = getTargets();
            if (targets.length === 0) { alert('삭제할 이미지가 없습니다.'); return; }
            if (!confirm(`총 ${targets.length}개의 이미지를 삭제합니다.`)) return;

            deleteBtn.disabled = true;
            deleteBtn.innerText = '열심히 지우는 중... ⏳';

            for (let i = 0; i < targets.length; i++) {
                const currentTargets = getTargets();
                if (currentTargets.length === 0) break;

                simulateRealClick(currentTargets[0]);

                let confirmBtn = null;
                for (let j = 0; j < 15; j++) {
                    await new Promise(r => setTimeout(r, 100));
                    confirmBtn = document.querySelector('.ant-modal-confirm-btns .ant-btn-primary') ||
                                 Array.from(document.querySelectorAll('button')).find(b => b.innerText.trim() === '삭제' || b.innerText.trim() === '확인');
                    if (confirmBtn && confirmBtn.offsetParent !== null) break;
                }

                if (confirmBtn) {
                    simulateRealClick(confirmBtn);
                    await new Promise(r => setTimeout(r, 800));
                }
            }
            deleteBtn.disabled = false;
            deleteBtn.innerText = '❌이미지 전체 삭제';
            alert('삭제 작업 완료! 화면에 [저장하기] 버튼이 있다면 꼭 누른 뒤 새로고침 하세요.');
        };

        // 3. 순서 번호 자동 채우기 (체크 버튼 저장 포함)
        const sortBtn = document.createElement('button');
        sortBtn.innerText = '🔢 순서 번호 자동 채우기';
        sortBtn.className = 'm-btn m-sort';
        sortBtn.onclick = async () => {
            const getEditButtons = () => Array.from(document.querySelectorAll('button')).filter(btn => {
                const svg = btn.querySelector('svg');
                return svg && svg.innerHTML.includes('M16.05 2.01');
            });

            let editButtons = getEditButtons();
            if (editButtons.length === 0) { alert('이름 수정(연필) 버튼을 찾지 못했습니다.'); return; }

            sortBtn.disabled = true;
            sortBtn.innerText = '번호 입력 중... ✍️';

            for (let i = 0; i < editButtons.length; i++) {
                let btn = getEditButtons()[i];
                if (!btn) continue;

                simulateRealClick(btn);
                await new Promise(r => setTimeout(r, 200));

                let input = document.activeElement;
                if (!input || input.tagName !== 'INPUT') {
                    const container = btn.closest('div[class*="flex"], div[class*="relative"]') || btn.parentElement.parentElement;
                    if(container) input = container.querySelector('input');
                }

                if (input && input.tagName === 'INPUT') {
                    const val = (i + 1).toString();
                    const setter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, "value").set;

                    if (setter) setter.call(input, val);
                    else input.value = val;

                    input.dispatchEvent(new Event('input', { bubbles: true }));
                    input.dispatchEvent(new Event('change', { bubbles: true }));

                    await new Promise(r => setTimeout(r, 100));

                    const saveBtns = Array.from(document.querySelectorAll('button')).filter(b => {
                        const svg = b.querySelector('svg');
                        return svg && svg.innerHTML.includes('m19.27 7.21-9.05');
                    });

                    if (saveBtns.length > 0) {
                        simulateRealClick(saveBtns[0]);
                    } else {
                        input.dispatchEvent(new KeyboardEvent('keydown', { bubbles: true, cancelable: true, key: 'Enter', code: 'Enter', keyCode: 13 }));
                        input.dispatchEvent(new Event('blur', { bubbles: true }));
                    }
                }
                await new Promise(r => setTimeout(r, 300));
            }

            sortBtn.disabled = false;
            sortBtn.innerText = '🔢 순서 번호 자동 채우기';
            alert(`총 ${editButtons.length}개의 이미지에 번호 입력 및 저장을 완료했습니다!`);
        };

        // 4. 빈 설명칸 "1" 채우기 (대사칸 제외)
        const fillTextBtn = document.createElement('button');
        fillTextBtn.innerText = '📝 빈 설명칸 "1" 채우기';
        fillTextBtn.className = 'm-btn m-fill';
        fillTextBtn.onclick = () => {
            const textareas = Array.from(document.querySelectorAll('textarea')).filter(ta => {
                const placeholder = ta.placeholder || "";
                const maxLength = ta.getAttribute('maxlength');

                if (placeholder.includes('뭐냥') || maxLength === '20') {
                    return false;
                }
                return placeholder.includes('고양이') || placeholder.includes('상황') || maxLength === '50';
            });

            if (textareas.length === 0) {
                alert('설명을 입력할 칸을 하나도 찾지 못했습니다.');
                return;
            }

            let filledCount = 0;
            textareas.forEach(ta => {
                if (!ta.value || ta.value.trim() === '') {
                    const setter = Object.getOwnPropertyDescriptor(window.HTMLTextAreaElement.prototype, "value").set;
                    if (setter) setter.call(ta, '1');
                    else ta.value = '1';

                    ta.dispatchEvent(new Event('input', { bubbles: true }));
                    ta.dispatchEvent(new Event('change', { bubbles: true }));
                    filledCount++;
                }
            });

            alert(`총 ${textareas.length}개의 설명칸 중 비어있던 ${filledCount}개에 '1'을 입력했습니다. (대사 칸은 제외됨)`);
        };

        container.appendChild(toggleBtn);
        container.appendChild(deleteBtn);
        container.appendChild(sortBtn);
        container.appendChild(fillTextBtn);
        document.body.appendChild(container);
    };

    // 🌟 핵심 감시자 (Observer) 🌟
    // 0.5초마다 주소를 확인하고 도구창이 지워졌으면 다시 살려냅니다.
    const loopObserver = () => {
        const isTargetPage = window.location.href.includes('/builder/story');
        const toolBox = document.getElementById(TOOL_ID);

        if (isTargetPage) {
            // 스토리 페이지인데 도구창이 사이트에 의해 삭제되었다면? -> 다시 생성!
            if (!toolBox && document.body) {
                renderUI();
            } else if (toolBox) {
                // 숨겨져 있다면 다시 보이게
                toolBox.style.display = 'flex';
            }
        } else {
            // 스토리 페이지가 아니면 도구창 숨기기
            if (toolBox) {
                toolBox.style.display = 'none';
            }
        }
    };

    // 무한 반복 감시 실행 (0.5초 간격)
    setInterval(loopObserver, 500);

})();
