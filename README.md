# jlu-jxpg-250609
2025年6月版的吉林大学教学评估一键评估脚本，全由Copilot书写，能跑

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

    // 设置数字输入框函数
    function setAntInputNumber(input, value) {
        try {
            console.log(`设置input值: ${value}`);
            
            // 1. 设置所有相关属性
            input.value = value.toString();
            input.defaultValue = value.toString();
            input.setAttribute('value', value.toString());
            input.setAttribute('aria-valuenow', value.toString());
            
            // 2. 触发聚焦
            input.focus();
            input.select();
            input.setSelectionRange(0, input.value.length);
            
            // 3. 发送删除键
            const deleteEvent = new KeyboardEvent('keydown', {
                bubbles: true,
                cancelable: true,
                key: 'Backspace',
                code: 'Backspace',
                keyCode: 8
            });
            input.dispatchEvent(deleteEvent);
            
            // 4. 清空值
            input.value = '';
            input.dispatchEvent(new Event('input', { bubbles: true }));
            
            // 5. 逐个字符输入新值
            const valueStr = value.toString();
            for (let i = 0; i < valueStr.length; i++) {
                const char = valueStr[i];
                
                // 键盘按下
                const keydownEvent = new KeyboardEvent('keydown', {
                    bubbles: true,
                    cancelable: true,
                    key: char,
                    code: char === '.' ? 'Period' : `Digit${char}`,
                    keyCode: char === '.' ? 190 : (48 + parseInt(char))
                });
                input.dispatchEvent(keydownEvent);
                
                // 更新值
                input.value = valueStr.substring(0, i + 1);
                input.setAttribute('value', input.value);
                input.setAttribute('aria-valuenow', input.value);
                
                // 输入事件
                const inputEvent = new Event('input', { bubbles: true, cancelable: true });
                Object.defineProperty(inputEvent, 'target', {
                    value: input,
                    enumerable: true
                });
                Object.defineProperty(inputEvent, 'data', {
                    value: char,
                    enumerable: true
                });
                input.dispatchEvent(inputEvent);
                
                // 键盘释放
                const keyupEvent = new KeyboardEvent('keyup', {
                    bubbles: true,
                    cancelable: true,
                    key: char,
                    code: char === '.' ? 'Period' : `Digit${char}`,
                    keyCode: char === '.' ? 190 : (48 + parseInt(char))
                });
                input.dispatchEvent(keyupEvent);
            }
            
            // 6. 最终事件
            input.dispatchEvent(new Event('change', { bubbles: true }));
            input.dispatchEvent(new Event('blur', { bubbles: true }));
            
            // 7. 强制更新React状态
            const nativeInputValueSetter = Object.getOwnPropertyDescriptor(
                window.HTMLInputElement.prototype,
                'value'
            ).set;
            nativeInputValueSetter.call(input, value.toString());
            
            const reactEvent = new Event('input', { bubbles: true });
            input.dispatchEvent(reactEvent);
            
            return true;
        } catch (error) {
            console.error('设置input失败:', error);
            return false;
        }
    }

    // 设置滑条UI函数
    function setSlider(container, value, maxValue) {
        try {
            // 查找相关元素
            const handle = container.querySelector('.index__slider-handle--q06om');
            const track = container.querySelector('.index__slider-track--lsz1P');
            const tooltip = handle?.querySelector('.index__slider-tooltip--on0rW');
            const sliderTooltip = container.querySelector('.index__sliderToolTip--vCPZF');
            
            if (!handle) {
                console.log('未找到滑条手柄');
                return false;
            }

            // 计算百分比位置
            const percentage = (value / maxValue) * 100;
            
            // 更新滑条视觉效果
            handle.style.left = `${percentage}%`;
            handle.style.borderColor = 'rgb(10, 200, 108)';
            
            if (track) {
                track.style.width = `${percentage}%`;
                if (percentage > 0) {
                    track.style.background = `linear-gradient(90deg, rgb(251, 156, 3) 0%, rgb(10, 200, 108) 100%)`;
                }
            }
            
            // 更新工具提示
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

            // 模拟滑条拖动事件
            const rail = container.querySelector('.index__slider-rail--Kr45T');
            if (rail) {
                const railRect = rail.getBoundingClientRect();
                const targetX = railRect.left + (railRect.width * percentage / 100);
                
                // 模拟鼠标事件
                const mouseDownEvent = new MouseEvent('mousedown', {
                    bubbles: true,
                    cancelable: true,
                    clientX: targetX,
                    clientY: railRect.top + railRect.height / 2,
                    button: 0
                });
                
                const mouseMoveEvent = new MouseEvent('mousemove', {
                    bubbles: true,
                    cancelable: true,
                    clientX: targetX,
                    clientY: railRect.top + railRect.height / 2,
                    button: 0
                });
                
                const mouseUpEvent = new MouseEvent('mouseup', {
                    bubbles: true,
                    cancelable: true,
                    clientX: targetX,
                    clientY: railRect.top + railRect.height / 2,
                    button: 0
                });
                
                rail.dispatchEvent(mouseDownEvent);
                rail.dispatchEvent(mouseMoveEvent);
                rail.dispatchEvent(mouseUpEvent);
            }
            
            return true;
        } catch (error) {
            console.error('设置滑条视觉效果失败:', error);
            return false;
        }
    }

    // 设置文本框函数
    function setTextArea(textarea, text) {
        try {
            textarea.focus();
            textarea.value = text;
            textarea.dispatchEvent(new Event('input', { bubbles: true }));
            textarea.dispatchEvent(new Event('change', { bubbles: true }));
            textarea.blur();
            return true;
        } catch (error) {
            console.error('设置文本框失败:', error);
            return false;
        }
    }

    // 处理模态框函数
    async function handleModal() {
        console.log('🔍 等待模态框出现...');
        
        // 等待模态框出现，最多等待5秒
        let modalFound = false;
        for (let i = 0; i < 50; i++) { // 50次 * 100ms = 5秒
            const modal = document.querySelector('.ant-modal-content');
            if (modal) {
                modalFound = true;
                console.log('✅ 发现模态框');
                break;
            }
            await sleep(100);
        }

        if (!modalFound) {
            console.log('⚠️ 未发现模态框，可能不需要处理');
            return;
        }

        await sleep(200); // 等待模态框完全加载

        // 查找模态框中的文本框
        const modalTextarea = document.querySelector('.ant-modal-content textarea');
        if (modalTextarea) {
            console.log('📝 填写模态框文本框...');
            setTextArea(modalTextarea, "好啊！");
            console.log('✅ 模态框文本框已填写');
        } else {
            console.log('❌ 未找到模态框文本框');
        }

        await sleep(300);

        // 点击确定按钮
        const confirmButton = document.querySelector('.ant-modal-content .ant-btn-primary');
        if (confirmButton) {
            console.log('🔘 点击确定按钮...');
            confirmButton.click();
            console.log('✅ 模态框已确认');
        } else {
            console.log('❌ 未找到确定按钮');
        }
    }

    console.log('🚀 开始自动填写教学评测...');

    try {
        // 1. 设置滑条值（降低延迟）
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
            
            // 设置数字输入框
            const inputSuccess = setAntInputNumber(input, targetValue);
            
            // 设置滑条UI
            const sliderSuccess = container ? setSlider(container, targetValue, maxValue) : true;
            
            await sleep(200); // 降低延迟从800ms到200ms
            
            console.log(`设置后值: "${input.value}"`);
            
            if (inputSuccess && sliderSuccess) {
                console.log(`✅ 第 ${i + 1} 个滑条完全设置成功`);
            } else {
                console.log(`❌ 第 ${i + 1} 个滑条设置有问题`);
            }
        }

        await sleep(300); // 降低延迟

        // 2. 选择正面标签
        const positiveCards = document.querySelectorAll('.index__card_item--tn687[style*="rgb(221, 245, 235)"]');
        console.log(`找到 ${positiveCards.length} 个正面标签`);
        
        for (let i = 0; i < Math.min(3, positiveCards.length); i++) {
            positiveCards[i].click();
            console.log(`✅ 选择标签: ${positiveCards[i].textContent}`);
            await sleep(100); // 降低延迟从200ms到100ms
        }

        await sleep(200); // 降低延迟

        // 3. 填写文本框
        const textAreas = document.querySelectorAll('textarea.index__UEditoTextarea--yga85');
        console.log(`找到 ${textAreas.length} 个文本框`);
        
        for (let textarea of textAreas) {
            const textSuccess = setTextArea(textarea, "很好很好！");
            if (textSuccess) {
                console.log('✅ 文本框设置成功');
            } else {
                console.log('❌ 文本框设置失败');
            }
            await sleep(100); // 降低延迟
        }

        await sleep(300); // 降低延迟

        // 4. 验证结果
        console.log('🔍 最终验证结果:');
        let allSuccess = true;
        
        numberInputs.forEach((input, index) => {
            const currentValue = parseFloat(input.value) || 0;
            console.log(`滑条 ${index + 1}: ${input.value}`);
            if (currentValue === 0) allSuccess = false;
        });
        
        textAreas.forEach((textarea, index) => {
            console.log(`文本框 ${index + 1}: "${textarea.value}"`);
            if (!textarea.value) allSuccess = false;
        });

        const selectedCards = document.querySelectorAll('.index__card_item--tn687[style*="rgb(42, 196, 121)"], .index__card_item--tn687[style*="rgb(10, 200, 108)"]');
        console.log(`已选择 ${selectedCards.length} 个标签`);

        // 5. 提交
        if (allSuccess) {
            console.log('🎉 所有内容设置成功！⏳ 1秒后自动提交...');
            await sleep(1000); // 降低延迟从3000ms到1000ms

            const submitButton = document.querySelector('button.index__submit--jiKIA') || 
                               document.querySelector('button.ant-btn-primary') ||
                               document.querySelector('button[class*="submit"]');
            
            if (submitButton) {
                submitButton.click();
                console.log('✅ 评测已提交！');
                
                // 6. 处理模态框
                await handleModal();
                
            } else {
                console.log('❌ 未找到提交按钮，请手动提交');
            }
        } else {
            console.log('⚠️ 部分内容设置失败，请检查后手动提交');
            console.log('💡 可以运行 manualSubmit() 手动提交');
        }

    } catch (error) {
        console.error('❌ 脚本执行出错:', error);
    }
}

// 手动提交函数（也包含模态框处理）
window.manualSubmit = async function() {
    const submitButton = document.querySelector('button.index__submit--jiKIA') || 
                       document.querySelector('button.ant-btn-primary');
    if (submitButton) {
        submitButton.click();
        console.log('✅ 已提交！');
        
        // 处理可能出现的模态框
        setTimeout(async () => {
            const modal = document.querySelector('.ant-modal-content');
            if (modal) {
                const modalTextarea = modal.querySelector('textarea');
                if (modalTextarea) {
                    modalTextarea.value = "好啊！";
                    modalTextarea.dispatchEvent(new Event('input', { bubbles: true }));
                }
                
                await new Promise(resolve => setTimeout(resolve, 300));
                
                const confirmButton = modal.querySelector('.ant-btn-primary');
                if (confirmButton) {
                    confirmButton.click();
                    console.log('✅ 模态框已处理');
                }
            }
        }, 1000);
        
    } else {
        console.log('❌ 未找到提交按钮');
    }
};

// 执行脚本
autoFillEvaluation();
```
