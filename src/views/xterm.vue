<template>
  <div id="xterm" class="xterm" />
</template>
<script>
import "xterm/css/xterm.css";
import { Terminal } from "xterm";
import { FitAddon } from "xterm-addon-fit";
import { AttachAddon } from "xterm-addon-attach";

export default {
  name: "Xterm",
  props: {
    socketURI: {
      type: String,
      default: "ws://127.0.0.1:8111/webssh"
    }
  },
  mounted() {
    this.initSocket();
  },
  beforeDestroy() {
    this.socket.close();
    this.term.dispose();
  },
  methods: {
    initTerm() {
      this.socket.send(
        JSON.stringify({
          operate: "connect",
          host: "", //IP
          port: "22", //端口号
          username: "root", //用户名
          password: "" //密码*/
        })
      );
      const term = new Terminal({
        rendererType: "canvas", //渲染类型
        rows: 40, //行数
        cols: 92, // 不指定行数，自动回车后光标从下一行开始
        convertEol: true, //启用时，光标将设置为下一行的开头
        // scrollback: 50, //终端中的回滚量
        disableStdin: false, //是否应禁用输入
        windowsMode: true, // 根据窗口换行
        // cursorStyle: "underline", //光标样式
        cursorBlink: true, //光标闪烁
        theme: {
          foreground: "#ECECEC", //字体
          background: "#000000", //背景色
          cursor: "help", //设置光标
          lineHeight: 20
        }
      });
      const attachAddon = new AttachAddon(this.socket);
      const fitAddon = new FitAddon();
      term.loadAddon(attachAddon);
      term.loadAddon(fitAddon);
      term.open(document.getElementById("xterm"));
      fitAddon.fit();
      term.focus();
      this.term = term;
      // 监视命令行输入
      this.term.onData(data => {
        let dataWrapper = data;
        if (dataWrapper === "\r") {
          dataWrapper = "\n";
        } else if (dataWrapper === "\u0003") {
          // 输入ctrl+c
          dataWrapper += "\n";
        }
        // 将输入的命令通知给后台，后台返回数据。
        this.socket.send(
          JSON.stringify({ operate: "command", command: dataWrapper })
        );
      });
      // window.addEventListener("resize", this.onTerminalResize);
    },
    initSocket() {
      this.socket = new WebSocket(this.socketURI);
      this.socketOnClose();
      this.socketOnOpen();
      this.socketOnError();
    },
    socketOnOpen() {
      this.socket.onopen = () => {
        // 链接成功后
        this.initTerm();
      };
    },
    socketOnClose() {
      this.socket.onclose = () => {
        // console.log('close socket')
      };
    },
    socketOnError() {
      this.socket.onerror = () => {
        // console.log('socket 链接失败')
      };
    }
  }
};
</script>
