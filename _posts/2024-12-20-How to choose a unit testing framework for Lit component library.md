

I recently selected a unit testing framework for the component library built with the company's Lit framework, and I have made some conclusions.

First of all, most of the Lit components on the Internet use the @open-wc/testing framework, but in actual use, I found that it is not very easy to use. The syntax is not very compatible with popular testing frameworks such as Jest, and the documentation is not very complete. You need to find some plug-ins to cooperate with it.

Finally, after comparison and analysis, I chose Vite's Vitest test framework. I will share the selection process with you.

### Automation testing concepts

Let's first understand some concepts of automated testing, and focus on the popular frameworks used for unit testing. You can get a clue from their creators and historical background. Many people think that test frameworks are similar. For example, the popular Mocha framework and Jest framework can be understood from their founding companies and founders, which engineering scenarios these two frameworks are suitable for.

![20241220-03.jpg]({{site.baseurl}}/assets/20241220-03.jpg)


### Selection

The selection should be considered from several perspectives, such as whether the documentation is complete, whether the community is active, how many people maintain the framework, how efficient the development is, and whether it is suitable for your project. Finally, our Lit framework component library is more suitable for unit testing using the Vitest test framework.


### Sample Code


{% highlight javascript %}

import { page, userEvent } from '@vitest/browser/context'
import { beforeEach, describe, expect, test } from "vitest"

import "../../index.css"
import "../checkbox"
import "../checkboxGroup/checkboxGroup"

async function render(html) {
    document.body.innerHTML = html;
    await new Promise(resolve => setTimeout(resolve, 0));
}

describe('Bee Checkbox', async () => {

    function getCheckboxEl(): HTMLElement {
        return document.querySelector('bee-checkbox')
    }

    function getCheckboxGroupEl(): HTMLElement {
        return document.querySelector('bee-checkbox-group')
    }

    test('create', async () => {
        await render('<bee-checkbox>腹部</bee-checkbox>')
        const checkbox = getCheckboxEl();
        const innerCheckbox = checkbox.querySelector('label');
        await userEvent.click(checkbox);
        expect(innerCheckbox.classList.contains("bee-checkbox--wrapper__checked")).toBe(true);
        await userEvent.click(checkbox);
        expect(innerCheckbox.classList.contains("bee-checkbox--wrapper__checked")).toBe(false);
    })

    test('disabled', async () => {
        await render('<bee-checkbox disabled>腹部</bee-checkbox>')
        const checkbox = getCheckboxEl();
        const innerCheckbox = checkbox.querySelector('label');
        expect(innerCheckbox.classList.contains("bee-checkbox__disabled")).toBe(true);
        await userEvent.click(checkbox);
        expect(innerCheckbox.classList.contains("checkbox__disabled")).toBe(false);
    })


    test('change', async () => {
        await render('<bee-checkbox>腹部</bee-checkbox>')
        const checkbox = getCheckboxEl();
        let checkData = null;
        checkbox.addEventListener('change', (e: CustomEvent) => {
            checkData = e.detail;
        })
        await userEvent.click(checkbox);
        expect(checkData).toBe(true)
    })

    describe('Bee Checkbox', async () => {
        test('checkbox group', async () => {
            await render(`<bee-checkbox-group>
            <bee-checkbox label="腹部"></bee-checkbox>
            <bee-checkbox label="胸部"></bee-checkbox>
            <bee-checkbox label="头部"></bee-checkbox>
            </bee-checkbox-group>`)
            const checkboxGroup = getCheckboxGroupEl();
            checkboxGroup['value'] = ['腹部'];
            const checkboxElement = document.querySelector('bee-checkbox[label="腹部"]');
            const innerCheckbox = checkboxElement.querySelector('label');
            await waitRender();
            expect(innerCheckbox.classList.contains("bee-checkbox--wrapper__checked")).toBe(true);
        })

        test('checkbox group min and max', async () => {
            await render(`<bee-checkbox-group min=1 max=3>
            <bee-checkbox label="腹部"></bee-checkbox>
            <bee-checkbox label="胸部"></bee-checkbox>
            <bee-checkbox label="头部"></bee-checkbox>
            <bee-checkbox label="足部"></bee-checkbox>
            </bee-checkbox-group>`)
            const checkboxGroup = getCheckboxGroupEl();
            checkboxGroup['value'] = ['腹部'];

            await userEvent.click(document.querySelector('bee-checkbox[label="腹部"]'));
            expect(checkboxGroup['value'].length).toBe(1);
            await userEvent.click(document.querySelector('bee-checkbox[label="胸部"]'));
            expect(checkboxGroup['value'].length).toBe(2);
            await userEvent.click(document.querySelector('bee-checkbox[label="头部"]'));
            expect(checkboxGroup['value'].length).toBe(3);
            await userEvent.click(document.querySelector('bee-checkbox[label="足部"]'));
            expect(checkboxGroup['value'].length).toBe(3);
        })

        test('checkbox group disabled', async () => {
            await render(`<bee-checkbox-group disabled>
            <bee-checkbox label="腹部"></bee-checkbox>
            <bee-checkbox label="胸部"></bee-checkbox>
            <bee-checkbox label="头部"></bee-checkbox>
            </bee-checkbox-group>`)
            const checkboxGroup = getCheckboxGroupEl();
            checkboxGroup['value'] = ['腹部','胸部','头部'];
            await userEvent.click(document.querySelector('bee-checkbox[label="腹部"]'));
            expect(checkboxGroup['value'].length).toBe(3);
            expect(checkboxGroup.firstElementChild.classList.contains("bee-checkbox-group__disabled")).toBe(true);
        })

        test('checkbox group marked', async () => {
            await render(`<bee-checkbox-group>
            <bee-checkbox label="腹部" marked></bee-checkbox>
            <bee-checkbox label="胸部"></bee-checkbox>
            <bee-checkbox label="头部"></bee-checkbox>
            </bee-checkbox-group>`)
            const checkboxGroup = getCheckboxGroupEl();
            checkboxGroup['value'] = ['腹部','胸部','头部'];
            const checkboxElement = document.querySelector('bee-checkbox[label="腹部"]');
            const innerCheckbox = checkboxElement.querySelector('label');
            expect(innerCheckbox.classList.contains("bee-checkbox__marked")).toBe(true);
        })

        test('checkbox group vertical', async () => {
            await render(`<bee-checkbox-group direction="vertical">
            <bee-checkbox label="腹部" marked></bee-checkbox>
            <bee-checkbox label="胸部"></bee-checkbox>
            <bee-checkbox label="头部"></bee-checkbox>
            </bee-checkbox-group>`)
            const checkboxGroup = getCheckboxGroupEl();
            expect(checkboxGroup.firstElementChild.classList.contains("bee-checkbox-group--wrapper__vertical")).toBe(true);
        })

    })
})

async function waitRender() {
    await new Promise(resolve => setTimeout(resolve, 0));
}

{% endhighlight %}
