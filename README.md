# jlu-jxpg-250609
2025年6月版的吉林大学教学评估一键评估脚本，全由Copilot书写，能跑

![jxpg250609.gif](https://s2.loli.net/2025/06/09/edmVvOy7nWh8c9l.gif)

## ⚠已知BUG

1. 当出现多个老师的课程（就是有多Tab）时，选择下一个老师后需要刷新页面，否则过完一遍会发现根本没提交

## 使用方法

1. 复制全部脚本
2. 进入具体的教学评估页面
3. F12或者右键检查打开开发者工具
4. 在Console/控制台窗口的输入框粘贴代码
5. 回车
6. 下一个老师同理，可以在Console/控制台输入框里按"↑"键来自动复制粘贴上一次执行过的代码
7. 回车，如此往复


直接复制下面的代码也可以，和index.js里面是一样的，就不用点进去了

```javascript
async function autoFillEvaluation() {
    function sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    // 增强的事件触发函数
    function triggerReactEvent(element, eventType, eventInit = {}) {
        const event = new Event(eventType, { bubbles: true, cancelable: true, ...eventInit });
        
        // 为React事件添加特殊属性
        Object.defineProperty(event, '_reactName', {
            value: `on${eventType.charAt(0).toUpperCase() + eventType.slice(1)}`,
            writable: false
        });
        
        // 尝试触发React的合成事件
        if (element._reactInternalFiber || element._reactInternalInstance) {
            const reactEvent = new Event(eventType, { bubbles: true });
            reactEvent.simulated = true;
            element.dispatchEvent(reactEvent);
        }
        
        element.dispatchEvent(event);
        return event;
    }

    // 强制更新React组件状态
    function forceReactUpdate(element) {
        try {
            // 查找React Fiber节点
            const reactFiber = element._reactInternalFiber || 
                              element._reactInternals || 
                              Object.keys(element).find(key => key.startsWith('__reactInternalInstance'));
            
            if (reactFiber) {
                // 尝试强制更新组件
                let fiber = typeof reactFiber === 'string' ? element[reactFiber] : reactFiber;
                while (fiber) {
                    if (fiber.stateNode && fiber.stateNode.forceUpdate) {
                        fiber.stateNode.forceUpdate();
                        break;
                    }
                    if (fiber.return) {
                        fiber = fiber.return;
                    } else {
                        break;
                    }
                }
            }
        } catch (error) {
            console.log('React更新失败，使用备用方案');
        }
    }

    // 改进的数字输入框设置函数
    function setAntInputNumber(input, value) {
        try {
            console.log(`设置input值: ${value}`);
            
            // 1. 先聚焦
            input.focus();
            
            // 2. 获取当前值并清空
            const currentValue = input.value;
            input.select();
            
            // 3. 使用execCommand删除选中内容（更接近真实用户操作）
            if (document.execCommand) {
                document.execCommand('delete', false, null);
            }
            
            // 4. 使用execCommand插入新值
            const valueStr = value.toString();
            if (document.execCommand) {
                document.execCommand('insertText', false, valueStr);
            } else {
                // 备用方案：逐字符输入
                input.value = '';
                for (let char of valueStr) {
                    const inputEvent = new InputEvent('input', {
                        bubbles: true,
                        cancelable: true,
                        inputType: 'insertText',
                        data: char
                    });
                    
                    input.value += char;
                    input.dispatchEvent(inputEvent);
                }
            }
            
            // 5. 强制设置所有相关属性
            input.value = valueStr;
            input.defaultValue = valueStr;
            input.setAttribute('value', valueStr);
            input.setAttribute('aria-valuenow', valueStr);
            
            // 6. 触发所有相关事件
            triggerReactEvent(input, 'input');
            triggerReactEvent(input, 'change');
            triggerReactEvent(input, 'blur');
            
            // 7. 强制React更新
            forceReactUpdate(input);
            
            // 8. 使用原生setter作为最后手段
            const nativeInputValueSetter = Object.getOwnPropertyDescriptor(
                window.HTMLInputElement.prototype, 'value'
            ).set;
            nativeInputValueSetter.call(input, valueStr);
            
            // 9. 最终验证
            setTimeout(() => {
                if (input.value !== valueStr) {
                    console.log(`重新设置input值: ${input.value} -> ${valueStr}`);
                    nativeInputValueSetter.call(input, valueStr);
                    triggerReactEvent(input, 'input');
                }
            }, 100);
            
            return true;
        } catch (error) {
            console.error('设置input失败:', error);
            return false;
        }
    }

    // 强化的文本框设置函数
    function setTextArea(textarea, text) {
        try {
            console.log(`🔧 强化设置textarea: "${text}"`);
            
            // 1. 聚焦并获取焦点
            textarea.focus();
            textarea.click(); // 确保获得焦点
            
            // 2. 选中所有内容并删除
            textarea.select();
            if (document.execCommand) {
                document.execCommand('selectAll', false, null);
                document.execCommand('delete', false, null);
            }
            
            // 3. 使用execCommand插入文本（最接近真实用户输入）
            if (document.execCommand) {
                const success = document.execCommand('insertText', false, text);
                console.log(`execCommand insertText 结果: ${success}`);
            }
            
            // 4. 备用方案：模拟键盘输入
            if (!textarea.value || textarea.value !== text) {
                console.log('使用备用输入方案');
                
                // 清空
                textarea.value = '';
                textarea.textContent = '';
                textarea.innerHTML = '';
                
                // 逐字符输入
                for (let i = 0; i < text.length; i++) {
                    const char = text[i];
                    const currentText = text.substring(0, i + 1);
                    
                    // 创建键盘事件
                    const keydownEvent = new KeyboardEvent('keydown', {
                        bubbles: true,
                        cancelable: true,
                        key: char,
                        code: char === ' ' ? 'Space' : `Key${char.toUpperCase()}`,
                        keyCode: char.charCodeAt(0)
                    });
                    textarea.dispatchEvent(keydownEvent);
                    
                    // 更新值
                    textarea.value = currentText;
                    textarea.textContent = currentText;
                    textarea.innerHTML = currentText;
                    
                    // 创建输入事件
                    const inputEvent = new InputEvent('input', {
                        bubbles: true,
                        cancelable: true,
                        inputType: 'insertText',
                        data: char
                    });
                    textarea.dispatchEvent(inputEvent);
                    
                    // 按键释放事件
                    const keyupEvent = new KeyboardEvent('keyup', {
                        bubbles: true,
                        cancelable: true,
                        key: char,
                        code: char === ' ' ? 'Space' : `Key${char.toUpperCase()}`,
                        keyCode: char.charCodeAt(0)
                    });
                    textarea.dispatchEvent(keyupEvent);
                }
            }
            
            // 5. 强制设置所有属性
            textarea.value = text;
            textarea.textContent = text;
            textarea.innerHTML = text;
            textarea.defaultValue = text;
            
            // 6. 使用原生setter
            const nativeTextAreaValueSetter = Object.getOwnPropertyDescriptor(
                window.HTMLTextAreaElement.prototype, 'value'
            ).set;
            nativeTextAreaValueSetter.call(textarea, text);
            
            // 7. 触发React事件
            triggerReactEvent(textarea, 'input');
            triggerReactEvent(textarea, 'change');
            
            // 8. 强制React更新
            forceReactUpdate(textarea);
            
            // 9. 创建持久化监控
            let persistenceAttempts = 0;
            const maxAttempts = 20; // 减少尝试次数
            
            const maintainValue = () => {
                if (persistenceAttempts >= maxAttempts) return;
                
                const currentValue = textarea.value || textarea.textContent || textarea.innerHTML;
                if (!currentValue || currentValue.trim() === '' || currentValue !== text) {
                    persistenceAttempts++;
                    console.log(`🔄 第${persistenceAttempts}次恢复textarea内容: "${text}"`);
                    
                    // 重新设置
                    nativeTextAreaValueSetter.call(textarea, text);
                    textarea.textContent = text;
                    textarea.innerHTML = text;
                    textarea.defaultValue = text;
                    
                    // 重新触发事件
                    triggerReactEvent(textarea, 'input');
                    triggerReactEvent(textarea, 'change');
                    forceReactUpdate(textarea);
                    
                    // 继续监控
                    setTimeout(maintainValue, 200);
                }
            };
            
            // 开始监控
            setTimeout(maintainValue, 200);
            
            // 10. MutationObserver作为额外保护
            const observer = new MutationObserver(() => {
                if (textarea.value !== text || textarea.textContent !== text) {
                    console.log('🔄 MutationObserver检测到变化，恢复内容');
                    nativeTextAreaValueSetter.call(textarea, text);
                    textarea.textContent = text;
                    textarea.innerHTML = text;
                }
            });
            
            observer.observe(textarea, { 
                attributes: true, 
                childList: true,
                characterData: true,
                subtree: true,
                attributeFilter: ['value']
            });
            
            // 15秒后停止监控
            setTimeout(() => {
                observer.disconnect();
                console.log(`⏰ 停止监控textarea`);
            }, 15000);
            
            // 失焦
            textarea.blur();
            
            console.log(`✅ textarea强化设置完成: "${textarea.value}"`);
            return true;
            
        } catch (error) {
            console.error('设置textarea失败:', error);
            return false;
        }
    }

    // 优化的模态框处理函数
    async function handleModal() {
        console.log('🔍 等待模态框出现...');
        
        let modalFound = false;
        for (let i = 0; i < 50; i++) {
            const modal = document.querySelector('.ant-modal-content');
            if (modal) {
                modalFound = true;
                console.log('✅ 发现模态框');
                break;
            }
            await sleep(100);
        }

        if (!modalFound) {
            console.log('⚠️ 未发现模态框');
            return;
        }

        await sleep(500); // 等待模态框完全渲染

        const modalTextarea = document.querySelector('.ant-modal-content textarea');
        if (modalTextarea) {
            console.log('📝 填写模态框文本框...');
            
            // 使用强化的设置方法
            modalTextarea.focus();
            modalTextarea.click();
            
            // 使用execCommand方式
            modalTextarea.select();
            if (document.execCommand) {
                document.execCommand('selectAll', false, null);
                document.execCommand('delete', false, null);
                const success = document.execCommand('insertText', false, '好啊！');
                console.log(`模态框execCommand结果: ${success}`);
            }
            
            // 强制设置
            const modalText = '好啊！';
            modalTextarea.value = modalText;
            modalTextarea.textContent = modalText;
            modalTextarea.innerHTML = modalText;
            modalTextarea.defaultValue = modalText;
            
            // 使用原生setter
            const nativeTextAreaValueSetter = Object.getOwnPropertyDescriptor(
                window.HTMLTextAreaElement.prototype, 'value'
            ).set;
            nativeTextAreaValueSetter.call(modalTextarea, modalText);
            
            // 触发事件
            triggerReactEvent(modalTextarea, 'input');
            triggerReactEvent(modalTextarea, 'change');
            forceReactUpdate(modalTextarea);
            
            // 持续监控模态框内容
            let modalCheckCount = 0;
            const modalCheck = setInterval(() => {
                if (modalCheckCount >= 15) { // 减少检查次数
                    clearInterval(modalCheck);
                    return;
                }
                
                const currentModal = document.querySelector('.ant-modal-content textarea');
                if (currentModal && (currentModal.value !== modalText || currentModal.textContent !== modalText)) {
                    console.log(`🔄 模态框内容检查第${modalCheckCount + 1}次，重新设置`);
                    nativeTextAreaValueSetter.call(currentModal, modalText);
                    currentModal.textContent = modalText;
                    currentModal.innerHTML = modalText;
                    triggerReactEvent(currentModal, 'input');
                }
                modalCheckCount++;
            }, 200);
            
            modalTextarea.blur();
            console.log(`✅ 模态框文本框设置完成: "${modalTextarea.value}"`);
        }

        await sleep(1000);

        const confirmButton = document.querySelector('.ant-modal-content .ant-btn-primary');
        if (confirmButton) {
            console.log('🔘 点击确定按钮...');
            confirmButton.click();
            console.log('✅ 模态框已确认');
            
            // 智能检测模态框关闭 - 改进版
            let modalClosed = false;
            for (let i = 0; i < 20; i++) { // 最多等待4秒
                await sleep(200);
                
                // 检查模态框是否真的关闭了
                const modalStillExists = document.querySelector('.ant-modal-content');
                const modalVisible = modalStillExists && 
                    modalStillExists.offsetParent !== null && 
                    getComputedStyle(modalStillExists).display !== 'none';
                
                // 检查页面是否已经跳转（URL变化或新内容出现）
                const urlChanged = window.location.href.includes('success') || 
                                 window.location.href !== window.location.href;
                const pageChanged = document.querySelector('.success') || 
                                  document.querySelector('[class*="success"]') ||
                                  document.title.includes('成功') ||
                                  document.title.includes('完成');
                
                if (!modalVisible || urlChanged || pageChanged) {
                    modalClosed = true;
                    console.log('🎉 模态框已成功关闭，评测完全完成！');
                    break;
                }
            }
            
            // 只有在确实没有关闭的情况下才提示
            if (!modalClosed) {
                const finalModal = document.querySelector('.ant-modal-content');
                if (finalModal && finalModal.offsetParent !== null) {
                    console.log('⚠️ 模态框可能仍然存在，请检查内容是否正确填写');
                    const stillModalTextarea = document.querySelector('.ant-modal-content textarea');
                    if (stillModalTextarea) {
                        console.log(`🔍 模态框仍存在，检查内容: "${stillModalTextarea.textContent || stillModalTextarea.value}"`);
                    }
                } else {
                    console.log('🎉 评测已完成！');
                }
            }
            
        } else {
            console.log('❌ 未找到确定按钮');
        }
    }

    // 设置滑条视觉效果的函数保持不变
    function setSlider(container, value, maxValue) {
        try {
            const handle = container.querySelector('.index__slider-handle--q06om');
            const track = container.querySelector('.index__slider-track--lsz1P');
            const tooltip = handle?.querySelector('.index__slider-tooltip--on0rW');
            const sliderTooltip = container.querySelector('.index__sliderToolTip--vCPZF');
            
            if (!handle) return false;

            const percentage = (value / maxValue) * 100;
            
            handle.style.left = `${percentage}%`;
            handle.style.borderColor = 'rgb(10, 200, 108)';
            
            if (track) {
                track.style.width = `${percentage}%`;
                if (percentage > 0) {
                    track.style.background = `linear-gradient(90deg, rgb(251, 156, 3) 0%, rgb(10, 200, 108) 100%)`;
                }
            }
            
            if (tooltip) {
                tooltip.textContent = value;
            }
            
            if (sliderTooltip) {
                sliderTooltip.style.left = `${percentage}%`;
                sliderTooltip.style.backgroundColor = 'rgb(10, 200, 108)';
                const tooltipText = sliderTooltip.firstChild;
                if (tooltipText && tooltipText.nodeType === Node.TEXT_NODE) {
                    tooltipText.textContent = value;
                }
                const triangle = sliderTooltip.querySelector('.index__triangle--ryIkF');
                if (triangle) {
                    triangle.style.borderColor = 'rgb(10, 200, 108) transparent transparent';
                }
            }
            
            return true;
        } catch (error) {
            console.error('设置滑条视觉效果失败:', error);
            return false;
        }
    }

    console.log('🚀 开始执行优化后的自动填写脚本...');

    try {
        // 1. 设置滑条值
        const sliderContainers = document.querySelectorAll('.index__sliderContainer--Bms8f');
        const numberInputs = document.querySelectorAll('.ant-input-number-input');
        console.log(`找到 ${sliderContainers.length} 个滑条容器，${numberInputs.length} 个数字输入框`);

        for (let i = 0; i < numberInputs.length; i++) {
            const input = numberInputs[i];
            const container = sliderContainers[i];
            
            const maxValue = parseInt(input.getAttribute('aria-valuemax')) || 
                           parseInt(input.getAttribute('max')) || 20;
            const targetValue = (i === numberInputs.length - 1) ? maxValue - 1 : maxValue;
            
            console.log(`设置第 ${i + 1} 个滑条: ${targetValue}/${maxValue}`);
            
            const inputSuccess = setAntInputNumber(input, targetValue);
            const sliderSuccess = container ? setSlider(container, targetValue, maxValue) : true;
            
            await sleep(300);
            
            console.log(`设置后值: "${input.value}"`);
        }

        await sleep(500);

        // 2. 选择正面标签
        const positiveCards = document.querySelectorAll('.index__card_item--tn687[style*="rgb(221, 245, 235)"]');
        console.log(`找到 ${positiveCards.length} 个正面标签`);
        
        for (let i = 0; i < Math.min(3, positiveCards.length); i++) {
            positiveCards[i].click();
            console.log(`✅ 选择标签: ${positiveCards[i].textContent}`);
            await sleep(200);
        }

        await sleep(500);

        // 3. 设置文本框
        const textAreas = document.querySelectorAll('textarea.index__UEditoTextarea--yga85');
        console.log(`找到 ${textAreas.length} 个文本框`);
        
        for (let textarea of textAreas) {
            console.log('🔧 开始设置文本框...');
            setTextArea(textarea, "很好很好！");
            await sleep(500);
        }

        // 4. 等待并验证 - 减少到1秒
        console.log('⏳ 等待1秒确保所有内容稳定...');
        await sleep(1000);

        // 5. 最终验证
        console.log('🔍 提交前最终验证:');
        let allGood = true;
        
        // 重新检查所有文本框
        const finalTextAreas = document.querySelectorAll('textarea.index__UEditoTextarea--yga85');
        finalTextAreas.forEach((textarea, index) => {
            const hasValue = textarea.value || textarea.textContent;
            console.log(`文本框 ${index + 1}: value="${textarea.value}", textContent="${textarea.textContent}"`);
            
            if (!hasValue || hasValue.trim() === '') {
                console.log(`❌ 文本框 ${index + 1} 仍为空，最后一次尝试设置...`);
                setTextArea(textarea, "很好很好！");
                allGood = false;
            }
        });

        if (!allGood) {
            console.log('⏳ 再等待1秒...');
            await sleep(1000);
        }

        // 6. 提交
        console.log('🚀 准备提交...');
        const submitButton = document.querySelector('button.index__submit--jiKIA') || 
                           document.querySelector('button.ant-btn-primary') ||
                           document.querySelector('button[class*="submit"]');
        
        if (submitButton) {
            console.log('🔘 点击提交按钮...');
            submitButton.click();
            console.log('✅ 评测已提交！');
            
            // 7. 处理模态框
            await handleModal();
            
        } else {
            console.log('❌ 未找到提交按钮');
        }

    } catch (error) {
        console.error('❌ 脚本执行出错:', error);
    }
}

// 执行脚本
autoFillEvaluation();
```
