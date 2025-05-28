# Web component

![Text Description automatically generated with medium confidence](../attachments/4008f84866506b4a3ad4168a9b3d5464.png)

![Graphical user interface, application Description automatically generated](../attachments/22a9e4755ab3e304c248bb9a83197063.png)

[https://github.com/web-padawan/awesome-lit\#general-resources](https://github.com/web-padawan/awesome-lit#general-resources)

Design System

<https://github.com/SAP/ui5-webcomponents>

<https://github.com/RedHat-UX/red-hat-design-system>

<https://github.com/shoelace-style/shoelace>

<https://github.com/vaadin/web-components>

## Slot

\<my-component\>

\<div\> Content1 \</div\>

\<div\> Content2 \</div\>

\<div slot = “name”\> Content3 \</div\>

\<div slot = “name”\> Content3 \</div\> //还可以多次使用同一个name的slot

\</my-component \>

在my-component的定义里面

\<div\>

\<slot\> Fallback \</slot\> Content1将出现的地方 如果没有指定，则显示Fallback

\<slot\>\</slot\> Content2 将出现的地方

\<slot name= ”name” \>\</slot\> Content3将出现的地方

\</div\>

## 自定义已封装的Component

<https://stackoverflow.com/questions/60266303/how-to-extend-and-style-existing-litelements>

import '@lion/button/define';

import {css} from 'lit-element'

import {LionButton} from '@lion/button'

class MyCustomButton extends LionButton {

static get styles() {

return [

super.styles,

css\`

**:host** {

background-color: red;

}

\`,

]

}

constructor() {

super()

}

}

customElements.define('my-button', MyCustomButton);

shadow dom的特性，外界的css无法影响到web component内部的css，防止css污染。

但可以通过配置解决，让外界可以污染。

## Shadow Dom

shadow dom最外面一层相当于父元素，传入的style（element style）优先级不如里面的高，所以里面定义的style外面改不了。

默认情况下外界的样式（比如使用class选择器）无法污染到web component里面的样式，需要进行配置？

<https://css-tricks.com/styling-a-web-component/>

可以在html或body、element选择器等， --custom 变量可以穿透

# Webpack

webpack可以将各种前端框架的源码（需要支持webpack的），打包为dist文件夹，然后直接放到nginx等web服务器wwwroot文件夹下进行host即部署完毕。

# 设计

## Diagrams.net

**Diagrams.net** (formerly draw.io) is a powerfull app designed to [create diagrams and flowcharts](https://www.diagrams.net/).

<https://roneo.org/en/diagrams.net-adding-icons-font-awesome-twemoji>
