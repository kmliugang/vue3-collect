<template>
    <div style="border: 1px solid #ccc">
        <Toolbar style="border-bottom: 1px solid #ccc" :editor="editorRef" :defaultConfig="toolbarConfig"
            :mode="mode" />
        <Editor style="height: 500px; overflow-y: hidden;" v-model="valueHtml" :defaultConfig="editorConfig"
            :mode="mode" @onCreated="handleCreated" @onChange="handleChange" @onDestroyed="handleDestroyed"
            @onFocus="handleFocus" @onBlur="handleBlur" @customAlert="customAlert" @customPaste="customPaste" />
    </div>
</template>
<script setup lang="ts">
// https://stackblitz.com/edit/vue3-wangeditor-demo-dodqjc?file=src%2Fcomponents%2FBasicEditor.vue
import '@/type/wangeditor.d.ts'
import '@wangeditor/editor/dist/css/style.css';
import { onBeforeUnmount, ref, shallowRef, onMounted, watch } from 'vue';
import { Editor, Toolbar } from '@wangeditor/editor-for-vue';

const emit = defineEmits(['select'])
// 编辑器实例，必须用 shallowRef
const editorRef = shallowRef()

// 内容 HTML
const valueHtml = ref('');

// 模拟 ajax 异步获取内容
onMounted(() => {
    setTimeout(() => {
        valueHtml.value = '<p>模拟 Ajax 异步设置内容</p>';
    }, 1500);
});

//配置功能栏
let toolbarConfig = {
    toolbarKeys: [
        'headerSelect',
        'blockquote',
        '|',
        'bold',
        'underline',
        'italic',
        {
            key: 'group-more-style',
            title: '更多',
            iconSvg:
                '<svg viewBox="0 0 1024 1024"><path d="M204.8 505.6m-76.8 0a76.8 76.8 0 1 0 153.6 0 76.8 76.8 0 1 0-153.6 0Z"></path><path d="M505.6 505.6m-76.8 0a76.8 76.8 0 1 0 153.6 0 76.8 76.8 0 1 0-153.6 0Z"></path><path d="M806.4 505.6m-76.8 0a76.8 76.8 0 1 0 153.6 0 76.8 76.8 0 1 0-153.6 0Z"></path></svg>',
            menuKeys: ['through', 'code', 'sup', 'sub']
        },
        'color',
        'bgColor',
        '|',
        'fontSize',
        {
            key: 'group-justify',
            title: '对齐',
            iconSvg:
                '<svg viewBox="0 0 1024 1024"><path d="M768 793.6v102.4H51.2v-102.4h716.8z m204.8-230.4v102.4H51.2v-102.4h921.6z m-204.8-230.4v102.4H51.2v-102.4h716.8zM972.8 102.4v102.4H51.2V102.4h921.6z"></path></svg>',
            menuKeys: ['justifyLeft', 'justifyRight', 'justifyCenter', 'justifyJustify']
        },
        'todo',
        'fontFamily',
        {
            key: 'group-indent',
            title: '缩进',
            iconSvg:
                '<svg viewBox="0 0 1024 1024"><path d="M0 64h1024v128H0z m384 192h640v128H384z m0 192h640v128H384z m0 192h640v128H384zM0 832h1024v128H0z m0-128V320l256 192z"></path></svg>',
            menuKeys: ['indent', 'delIndent']
        },
        '|',
        'emotion',
        'insertLink',
        'uploadImage',
        'insertTable',
        'codeBlock',
        'divider',
        'clearStyle',
        '|',
        'undo',
        'redo',
    ]
}


const uploadImageList = ref([])
const saveImageList = ref([])
const mode = ref('default');

//上传本地图片
function update(file: any, insertFn: (params: string) => void) {
    console.log(file, insertFn);

    const reader = new FileReader();
    reader.readAsDataURL(file)
    reader.onload = function (e: any) {
        insertFn(e.target.result)
    }
}

function getOnInsertedImage() {
    uploadImageList.value.push()
}

//编辑器配置
let editorConfig = {
    placeholder: '请输入内容...',
    // 所有的菜单配置，都要在 MENU_CONF 属性下
    MENU_CONF: {
        insertImage: {
            onInsertedImage: getOnInsertedImage()
        },
        // 配置上传图片
        uploadImage: {
            customUpload: update
        }
    }
}


// 组件销毁时，也及时销毁编辑器
onBeforeUnmount(() => {
    const editor = editorRef.value
    if (editor == null) return
    editor.destroy()
})


// function copyObject(obj) {
//     return JSON.parse(JSON.stringify(obj));
// }
const handleCreated = (editor:any) => {
    editorRef.value = editor // 记录 editor 实例，重要！
    // saveImageList.value = editor.getElemsByType('image')
    // uploadImageList.value = copyObject(saveImageList.value)
    // console.log('created', editor)
}

watch(() => valueHtml.value, () => {
    //当编辑器的内容发生变化时，把值传给父组件
    emit('select', valueHtml.value)
})


const handleChange:EditorEvent = (editor) => { console.log('change:', editor.children) }
const handleDestroyed:EditorEvent = (editor) => { console.log('destroyed', editor) }
const handleFocus:EditorEvent = (editor) => { console.log('focus', editor) }
const handleBlur:EditorEvent = (editor) => { console.log('blur', editor) }
const customAlert:EditorMethodString = (info, type) => { alert(`【自定义提示】${type} - ${info}`) }
const customPaste:EditorMethod = (editor, event, callback) => {
    console.log('ClipboardEvent 粘贴事件对象', event)
    // const html = event.clipboardData.getData('text/html') // 获取粘贴的 html
    // const text = event.clipboardData.getData('text/plain') // 获取粘贴的纯文本
    // const rtf = event.clipboardData.getData('text/rtf') // 获取 rtf 数据（如从 word wsp 复制粘贴）

    // 自定义插入内容
    editor.insertText('xxx')

    // 返回 false ，阻止默认粘贴行为
    event.preventDefault()
    callback(false) // 返回值（注意，vue 事件的返回值，不能用 return）

    // 返回 true ，继续默认的粘贴行为
    // callback(true)
}

//父组件调用子组件的方法清空编辑器内容
const abc = function () {
    valueHtml.value = ''
}

</script>

<style scoped></style>