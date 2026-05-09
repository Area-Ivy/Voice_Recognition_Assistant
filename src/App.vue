<template>
  <div class="app-wrapper">
    <!-- 顶栏 -->
    <header class="app-header">
      <h1>{{ pageTitle }}</h1>
    </header>

    <!-- 主要内容区 -->
    <el-container class="main-container">
      <!-- 左侧控制台 -->
      <el-aside class="control-sidebar">
        <!-- 状态和控制按钮 -->
        <el-card class="control-card">
          <template #header>
            <div class="card-header">
              <h3>语音控制</h3>
            </div>
          </template>
          
          <el-alert
            :title="statusMessage"
            :type="isListening ? 'success' : 'info'"
            :closable="false"
            center
            class="status-alert"
          />
          <div class="control-buttons">
            <div class="custom-button primary-btn" @click="startListening" :class="{ disabled: isListening }">
              <i class="icon"><el-icon><Microphone /></el-icon></i>
              <span class="text">开始识别</span>
            </div>
            <div class="custom-button danger-btn" @click="stopListening" :class="{ disabled: !isListening }">
              <i class="icon"><el-icon><Mute /></el-icon></i>
              <span class="text">停止识别</span>
            </div>
            <div class="custom-button default-btn" @click="toggleTheme">
              <i class="icon"><el-icon><Switch /></el-icon></i>
              <span class="text">切换主题</span>
            </div>
          </div>
        </el-card>

        <!-- 功能按钮 -->
        <el-card class="function-card">
          <template #header>
            <div class="card-header">
              <h3>功能操作</h3>
            </div>
          </template>
          
          <el-dropdown trigger="click" @command="handleCommand" class="function-dropdown">
            <el-button type="primary" class="dropdown-button">
              功能菜单
              <el-icon class="el-icon--right"><arrow-down /></el-icon>
            </el-button>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item-group title="文本操作">
                  <el-dropdown-item :icon="Document" command="clear">清除文本</el-dropdown-item>
                  <el-dropdown-item :icon="Download" command="save">保存文本</el-dropdown-item>
                </el-dropdown-item-group>
                <el-dropdown-item-group title="音乐控制">
                  <el-dropdown-item :icon="VideoPlay" command="play">播放音乐</el-dropdown-item>
                  <el-dropdown-item :icon="VideoPause" command="pause">暂停音乐</el-dropdown-item>
                  <el-dropdown-item :icon="VideoPlay" command="resume">继续播放</el-dropdown-item>
                  <el-dropdown-item :icon="MuteNotification" command="stop">停止播放</el-dropdown-item>
                  <el-dropdown-item :icon="ArrowLeft" command="prev">上一首</el-dropdown-item>
                  <el-dropdown-item :icon="ArrowRight" command="next">下一首</el-dropdown-item>
                  <el-dropdown-item :icon="Plus" command="volumeUp">音量+</el-dropdown-item>
                  <el-dropdown-item :icon="Minus" command="volumeDown">音量-</el-dropdown-item>
                </el-dropdown-item-group>
                <el-dropdown-item-group title="转写模式">
                  <el-dropdown-item :icon="Edit" command="transcribe">
                    {{ isTranscribing ? '退出转写模式' : '进入转写模式' }}
                  </el-dropdown-item>
                </el-dropdown-item-group>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          
          <!-- 指令日志下拉菜单 -->
          <el-dropdown trigger="click" class="function-dropdown" style="margin-top: 6px;">
            <el-button type="info" class="dropdown-button">
              指令日志
              <el-icon class="el-icon--right"><arrow-down /></el-icon>
            </el-button>
            <template #dropdown>
              <el-dropdown-menu v-if="commandLogs.length">
                <el-dropdown-item-group title="最近指令">
                  <el-dropdown-item 
                    v-for="(log, index) in commandLogs" 
                    :key="`cmd-${index}`"
                    :icon="Document"
                    :class="{ 'is-active': log.type === 'current' }"
                  >
                    {{ log.text }}
                  </el-dropdown-item>
                </el-dropdown-item-group>
              </el-dropdown-menu>
              <el-dropdown-menu v-else>
                <el-dropdown-item disabled>暂无指令记录</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          
          <!-- 历史记录下拉菜单 -->
          <el-dropdown trigger="click" class="function-dropdown" style="margin-top: 6px;">
            <el-button type="success" class="dropdown-button">
              历史记录
              <el-icon class="el-icon--right"><arrow-down /></el-icon>
            </el-button>
            <template #dropdown>
              <el-dropdown-menu v-if="savedSessions.length">
                <el-dropdown-item-group title="历史内容">
                  <div v-for="(session, index) in savedSessions" :key="`ses-${index}`">
                    <el-dropdown-item :icon="Document">
                      <div class="history-dropdown-item">
                        <span class="history-preview" :title="session.fullText">{{ session.preview }}</span>
                        <el-button
                          type="primary"
                          size="small"
                          @click.stop="restoreSession(index)"
                          :icon="DocumentCopy"
                        >
                          恢复
                        </el-button>
                      </div>
                    </el-dropdown-item>
                  </div>
                </el-dropdown-item-group>
              </el-dropdown-menu>
              <el-dropdown-menu v-else>
                <el-dropdown-item disabled>暂无历史记录</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          
          <!-- 使用说明下拉菜单 -->
          <el-dropdown trigger="click" class="function-dropdown" style="margin-top: 6px;">
            <el-button type="warning" class="dropdown-button">
              使用说明
              <el-icon class="el-icon--right"><arrow-down /></el-icon>
            </el-button>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item-group title="语音指令说明">
                  <el-dropdown-item :icon="VideoPlay">
                    <span class="help-command">"播放音乐"</span> - 开始播放音乐
                  </el-dropdown-item>
                  <el-dropdown-item :icon="VideoPause">
                    <span class="help-command">"暂停音乐"</span> - 暂停当前播放
                  </el-dropdown-item>
                  <el-dropdown-item :icon="VideoPlay">
                    <span class="help-command">"继续播放"</span> - 继续播放音乐
                  </el-dropdown-item>
                  <el-dropdown-item :icon="MuteNotification">
                    <span class="help-command">"停止播放"</span> - 停止并关闭播放器
                  </el-dropdown-item>
                  <el-dropdown-item :icon="ArrowLeft">
                    <span class="help-command">"上一首"</span> - 播放上一首歌曲
                  </el-dropdown-item>
                  <el-dropdown-item :icon="ArrowRight">
                    <span class="help-command">"下一首"</span> - 播放下一首歌曲
                  </el-dropdown-item>
                </el-dropdown-item-group>
                <el-dropdown-item-group title="音量控制">
                  <el-dropdown-item :icon="Plus">
                    <span class="help-command">"调大音量"</span> 或 <span class="help-command">"大声一点"</span> - 增加音量
                  </el-dropdown-item>
                  <el-dropdown-item :icon="Minus">
                    <span class="help-command">"调小音量"</span> 或 <span class="help-command">"小声一点"</span> - 降低音量
                  </el-dropdown-item>
                </el-dropdown-item-group>
                <el-dropdown-item-group title="文本操作">
                  <el-dropdown-item :icon="Document">
                    <span class="help-command">"清除文本"</span> - 清除并归档当前文本
                  </el-dropdown-item>
                  <el-dropdown-item :icon="Download">
                    <span class="help-command">"保存文本"</span> - 将当前文本导出为文件
                  </el-dropdown-item>
                </el-dropdown-item-group>
                <el-dropdown-item-group title="转写模式">
                  <el-dropdown-item :icon="Edit">
                    <span class="help-command">"转文字"</span> - 进入转写模式
                  </el-dropdown-item>
                  <el-dropdown-item :icon="Edit">
                    <span class="help-command">"结束"</span> 或 <span class="help-command">"退出"</span> - 退出转写模式
                  </el-dropdown-item>
                </el-dropdown-item-group>
                <el-dropdown-item-group title="其他功能">
                  <el-dropdown-item :icon="Switch">
                    <span class="help-command">"切换主题"</span> - 切换明暗主题
                  </el-dropdown-item>
                </el-dropdown-item-group>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </el-card>
      </el-aside>

      <!-- 右侧主要内容区 -->
      <el-container class="right-content">
        <!-- 转写内容区域 -->
        <el-main class="transcription-area">
          <el-card class="transcription-card">
            <template #header>
              <div class="card-header">
                <h3>转写内容</h3>
                <div class="button-group">
                  <el-button type="primary" @click="clearAndArchiveNote" :icon="Document">
                    完成并归档
                  </el-button>
                  <el-button type="success" @click="saveNoteToFile" :icon="Download">
                    导出为 TXT
                  </el-button>
                </div>
              </div>
            </template>
            <el-input
              type="textarea"
              v-model="notepadContent"
              :rows="8"
              resize="vertical"
              placeholder="转文字模式下的语音识别结果会显示在这里..."
            />
          </el-card>
        </el-main>
        
        <!-- 音乐播放器 -->
        <el-aside class="player-area" v-show="isMusicPlayerActive">
          <el-card class="music-player-card">
            <template #header>
              <div class="card-header">
                <h3>音乐播放器</h3>
              </div>
            </template>
            
            <div class="music-player-content">
              <el-image
                v-if="currentSong"
                :src="currentSong.cover"
                :alt="currentSong.title"
                class="cover-image"
                fit="cover"
              />
              <div class="song-info">{{ songTitle }}</div>
              <div class="player-controls">
                <el-button :icon="ArrowLeft" circle @click="playPreviousSong" />
                <el-button 
                  :icon="playerIsPlaying ? VideoPause : VideoPlay"
                  type="primary"
                  circle
                  @click="currentSongAction"
                />
                <el-button :icon="ArrowRight" circle @click="playNextSong" />
              </div>
            </div>
            
            <audio
              ref="playerElement"
              @ended="playNextSong"
              @play="playerIsPlaying = true"
              @pause="playerIsPlaying = false"
            />
          </el-card>
        </el-aside>
      </el-container>
    </el-container>
    
    <!-- 底栏 -->
    <footer class="app-footer">
      <div class="footer-content">
        <p>© 2025 语音识别与控制系统 | 版本 1.0.0</p>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted, watch } from 'vue';
import {
  Microphone,
  Mute,
  Switch,
  Document,
  Download,
  DocumentCopy,
  VideoPlay,
  VideoPause,
  ArrowLeft,
  ArrowRight,
  Edit,
  Plus,
  Minus,
  MuteNotification,
  ArrowDown
} from '@element-plus/icons-vue';

const pageTitle = ref('中文语音识别与控制系统');
const theme = ref('light');
const statusMessage = ref('状态：空闲中');
const commandLogs = ref([]);
const notepadContent = ref('');
const savedSessions = ref([]);
const isListening = ref(false);
const isTranscribing = ref(false);
let recognition = null;
const synth = window.speechSynthesis;
let voices = [];
const playerElement = ref(null);
const isMusicPlayerActive = ref(false);
const playerIsPlaying = ref(false);
const currentSongIndex = ref(0);
const playlist = reactive([
  { title: 'Home Sweet Home', file: 'music/song1.mp3', cover: 'images/song1.jpg' },
  { title: 'Everytime', file: 'music/song2.mp3', cover: 'images/song2.png' },
  { title: '링가링가', file: 'music/song3.mp3', cover: 'images/song3.jpg' }
]);

const currentSong = computed(() => playlist[currentSongIndex.value]);
const songTitle = computed(() => currentSong.value ? `当前歌曲：${currentSong.value.title}` : '没有选择歌曲');

const loadVoices = () => {
  voices = synth.getVoices().filter(voice => voice.lang.includes('zh'));
  if (voices.length === 0) {
      voices = synth.getVoices().filter(voice => voice.lang.includes('zh'));
  }
};

const speak = (text) => {
  if (!synth) { return; }
  if (voices.length === 0) { setTimeout(() => speak(text), 100); return; }
  if (synth.speaking) { synth.cancel(); }
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.voice = voices.find(v => v.name.includes('Google') && v.lang.includes('zh')) || voices.find(v => v.lang.includes('zh'));
  utterance.lang = 'zh-CN';
  utterance.onerror = (event) => console.error('语音播报发生错误:', event);
  synth.speak(utterance);
};

const logCommand = (text) => {
  const currentLog = commandLogs.value.find(log => log.type === 'current');
  if (currentLog) {
    currentLog.type = 'executed';
    currentLog.text = currentLog.text.replace(" - 当前执行", "");
  }
  commandLogs.value.unshift({ text: `${text} - 当前执行`, type: 'current' });
};

const handleVoiceCommand = (text) => {
    logCommand(`指令: ${text}`);
    if (text.includes("清除文本")) { clearAndArchiveNote(); } 
    else if (text.includes("保存文本")) { saveNoteToFile(); }
    else if (text.includes("播放音乐")) { playMusic(); }
    else if (text.includes("暂停音乐")) { pauseMusic(); }
    else if (text.includes("继续播放")) { resumeMusic(); }
    else if (text.includes("停止播放")) { stopMusic(); }
    else if (text.includes("下一首")) { playNextSong(); }
    else if (text.includes("上一首")) { playPreviousSong(); }
    else if (text.includes("调大音量") || text.includes("大声一点")) { changeVolume('up'); }
    else if (text.includes("调小音量") || text.includes("小声一点")) { changeVolume('down'); }
    else if (text.includes("切换主题")) { toggleTheme(); }
    else { commandLogs.value.shift(); statusMessage.value = "未识别的指令：" + text; }
};

const clearAndArchiveNote = () => {
  const contentToSave = notepadContent.value.trim();
  if (contentToSave) {
    savedSessions.value.unshift({
      preview: contentToSave.substring(0, 20) + (contentToSave.length > 20 ? '...' : ''),
      fullText: contentToSave
    });
    speak("内容已归档");
  }
  notepadContent.value = "";
};

const saveNoteToFile = () => {
  if (!notepadContent.value.trim()) { speak("没有内容可以导出。"); return; }
  const blob = new Blob([notepadContent.value], { type: "text/plain;charset=utf-8" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "语音记事.txt";
  link.click();
  URL.revokeObjectURL(link.href);
  speak("转写内容已导出为文件。");
};

const restoreSession = (index) => {
  const sessionToRestore = savedSessions.value[index];
  if (sessionToRestore) {
    if (notepadContent.value.trim()) { clearAndArchiveNote(); }
    notepadContent.value = sessionToRestore.fullText;
    speak("内容已恢复");
  }
};

const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light';
  speak(`已切换到${theme.value === 'light' ? '白天' : '夜间'}模式`);
  
  // 更新页面标题以反映主题变化
  pageTitle.value = `中文语音识别与控制系统 - ${theme.value === 'light' ? '白天' : '夜间'}模式`;
};

const switchSong = (index) => {
  if (index >= 0 && index < playlist.length && playerElement.value) {
    currentSongIndex.value = index;
    const song = currentSong.value;
    playerElement.value.src = song.file;
    playerElement.value.play();
    isMusicPlayerActive.value = true;
    speak(`正在播放, ${song.title}`);
  }
};

const currentSongAction = () => {
    if(playerIsPlaying.value) { playerElement.value.pause(); }
    else if (playerElement.value) { playerElement.value.play(); }
};

const playNextSong = () => { switchSong((currentSongIndex.value + 1) % playlist.length); };

const playPreviousSong = () => { switchSong((currentSongIndex.value - 1 + playlist.length) % playlist.length); };

const changeVolume = (direction) => {
    if (!playerElement.value) return;
    let v = playerElement.value.volume;
    v = direction === 'up' ? Math.min(1, v + 0.2) : Math.max(0, v - 0.2);
    playerElement.value.volume = v;
    speak(`音量已调整到百分之 ${Math.round(v * 100)}`);
};

const startListening = () => {
  if (isListening.value) return;
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  if (!SpeechRecognition) { alert("抱歉，您的浏览器不支持 Web Speech API。"); return; }
  
  recognition = new SpeechRecognition();
  recognition.continuous = true;
  recognition.interimResults = true;
  recognition.lang = 'zh-CN';

  recognition.onstart = () => { isListening.value = true; statusMessage.value = "状态：🎙 正在识别中..."; };
  recognition.onend = () => { isListening.value = false; statusMessage.value = "状态：空闲中"; recognition = null; };
  recognition.onerror = (event) => { console.error('语音识别错误:', event.error); statusMessage.value = `识别错误: ${event.error}`; };

  recognition.onresult = (event) => {
    let final_transcript = '';
    for (let i = event.resultIndex; i < event.results.length; ++i) {
        if (event.results[i].isFinal) {
            final_transcript += event.results[i][0].transcript;
        }
    }
    
    final_transcript = final_transcript.trim();
    if (!final_transcript) return;
    
    if (isTranscribing.value) {
      if (/^(结束|退出)/.test(final_transcript)) {
        isTranscribing.value = false;
        logCommand("模式切换: 结束转文字");
        speak("好的，已退出转文字模式");
        statusMessage.value = "状态：🎙 正在识别中...";
        return;
      }
      notepadContent.value += final_transcript + "\n";
    } else {
      if (/^转文字/.test(final_transcript)) {
        isTranscribing.value = true;
        logCommand("模式切换: 转文字");
        speak("已进入转文字模式");
        statusMessage.value = "状态：转文字模式中 (请说话)";
        return;
      }
      handleVoiceCommand(final_transcript);
    }
  };

  recognition.start();
};

const stopListening = () => {
  if (recognition && isListening.value) {
    isListening.value = false;
    recognition.stop();
  }
};

const toggleTranscribeMode = () => {
  isTranscribing.value = !isTranscribing.value;
  if (isTranscribing.value) {
    logCommand("模式切换: 转文字");
    speak("已进入转文字模式");
    statusMessage.value = "状态：转文字模式中 (请说话)";
  } else {
    logCommand("模式切换: 结束转文字");
    speak("好的，已退出转文字模式");
    statusMessage.value = isListening.value ? "状态：🎙 正在识别中..." : "状态：空闲中";
  }
};

// 添加音乐控制函数
const playMusic = () => {
  switchSong(currentSongIndex.value);
};

const pauseMusic = () => {
  if (playerElement.value) { 
    playerElement.value.pause(); 
    speak(`已暂停`); 
  }
};

const resumeMusic = () => {
  if (playerElement.value && playerElement.value.paused) { 
    playerElement.value.play(); 
    speak('继续播放'); 
  }
};

const stopMusic = () => {
  if (playerElement.value) { 
    playerElement.value.pause(); 
    playerElement.value.currentTime = 0; 
    isMusicPlayerActive.value = false; 
    speak('已停止播放'); 
  }
};

// 添加下拉菜单命令处理函数
const handleCommand = (command) => {
  switch (command) {
    case 'clear':
      clearAndArchiveNote();
      break;
    case 'save':
      saveNoteToFile();
      break;
    case 'play':
      playMusic();
      break;
    case 'pause':
      pauseMusic();
      break;
    case 'resume':
      resumeMusic();
      break;
    case 'stop':
      stopMusic();
      break;
    case 'prev':
      playPreviousSong();
      break;
    case 'next':
      playNextSong();
      break;
    case 'volumeUp':
      changeVolume('up');
      break;
    case 'volumeDown':
      changeVolume('down');
      break;
    case 'transcribe':
      toggleTranscribeMode();
      break;
  }
};

onMounted(() => {
  document.title = pageTitle.value;
  if (synth.onvoiceschanged !== undefined) {
    synth.onvoiceschanged = loadVoices;
  }
  loadVoices();

  const savedNote = localStorage.getItem('notepadContent');
  if (savedNote) notepadContent.value = savedNote;
  
  const savedHistory = localStorage.getItem('savedSessions');
  if(savedHistory) savedSessions.value = JSON.parse(savedHistory);

  const savedTheme = localStorage.getItem('appTheme');
  if (savedTheme) theme.value = savedTheme;

  const savedSongIndex = localStorage.getItem('currentSongIndex');
  if (savedSongIndex) currentSongIndex.value = parseInt(savedSongIndex, 10);
});

onUnmounted(() => {
  if (recognition) {
    recognition.stop();
  }
});

watch(notepadContent, (newValue) => { localStorage.setItem('notepadContent', newValue); });

watch(savedSessions, (newSessions) => {
    localStorage.setItem('savedSessions', JSON.stringify(newSessions));
}, { deep: true });

watch(currentSongIndex, (newIndex) => { localStorage.setItem('currentSongIndex', newIndex); });

watch(theme, (newTheme, oldTheme) => {
  localStorage.setItem('appTheme', newTheme);
  if(oldTheme) {
      document.body.classList.remove(oldTheme);
  }
  document.body.classList.add(newTheme);
}, { immediate: true });
</script>

<style>
@import 'element-plus/dist/index.css';

:root {
  --primary-blue: #409EFF;
  --primary-dark: #337ecc;
  --primary-light: #ECF5FF;
  --text-primary: #2C3E50;
  --text-secondary: #606266;
  --border-color: #DCDFE6;
  --hover-color: #79BBFF;
  --bg-light: #F5F7FA;
  --card-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  
  /* 从style.css迁移的变量 */
  font-family: 'PingFang SC', 'Microsoft YaHei', Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body.light {
  --bg-color: var(--bg-light);
  --text-color: var(--text-primary);
  --header-color: #ffffff;
  --button-bg: var(--primary-blue);
  --button-primary: var(--primary-blue);
  --button-success: #67c23a;
  --button-danger: #f56c6c;
  --button-info: #909399;
  --card-bg: #ffffff;
  --card-title-color: var(--text-primary);
  --header-bg: var(--primary-blue);
  --footer-bg: #333333;
  --footer-text: #cccccc;
}

body.dark {
  --bg-color: #1e1e1e;
  --text-color: #ffffff;
  --header-color: #ffffff;
  --button-bg: #409EFF;
  --button-primary: #0d5aa7;
  --button-success: #2d6324;
  --button-danger: #a93131;
  --button-info: #4a4c50;
  --card-bg: #2b2b2b;
  --card-title-color: #ffffff;
  --header-bg: #1a1a1a;
  --footer-bg: #1a1a1a;
  --footer-text: #999999;
}

/* 链接样式 */
a {
  font-weight: 500;
  color: var(--primary-blue);
  text-decoration: inherit;
}
a:hover {
  color: var(--hover-color);
}

body {
  font-family: 'PingFang SC', 'Microsoft YaHei', Arial, sans-serif;
  background-color: var(--bg-color);
  color: var(--text-color);
  margin: 0;
  padding: 0;
  min-height: 100vh;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  transition: background-color 0.3s ease, color 0.3s ease;
}

/* 应用布局 */
.app-wrapper {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  width: 100%;
}

/* 顶栏样式 */
.app-header {
  background-color: var(--header-bg);
  padding: 16px 24px;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 100%;
  left: 0;
  right: 0;
  transition: background-color 0.3s ease;
}

.app-header h1 {
  color: var(--header-color);
  font-size: 24px;
  margin: 0;
  font-weight: 600;
}

/* 主要内容区样式 */
.main-container {
  flex: 1;
  display: flex;
  width: 100%;
}

/* 左侧边栏样式 */
.control-sidebar {
  width: 300px !important;
  background: var(--card-bg);
  border-right: 1px solid var(--border-color);
  padding: 20px;
  flex-shrink: 0;
  transition: background-color 0.3s ease;
}

.control-card, .function-card, .transcription-card, .music-player-card, .history-card {
  margin-bottom: 20px;
  transition: background-color 0.3s ease, border-color 0.3s ease;
  color: var(--text-color);
}

.el-card :deep(.el-card__header) {
  color: var(--card-title-color);
  transition: color 0.3s ease;
}

.song-info, .history-preview {
  color: var(--text-color);
  transition: color 0.3s ease;
}

.function-card :deep(.el-card__header) {
  padding: 15px 20px;
}

.function-dropdown {
  width: 100%;
  margin-bottom: 6px;
}

.function-dropdown:last-child {
  margin-bottom: 0;
}

.dropdown-button {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 15px;
  height: 36px;
  transition: background-color 0.3s, border-color 0.3s;
}

.quick-actions {
  margin-top: 10px;
}

.function-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 16px;
}

.function-buttons .el-button {
  flex: 1 0 calc(50% - 5px);
  display: flex;
  justify-content: center;
  align-items: center;
  max-width: calc(50% - 5px);
  margin: 0;
}

.control-buttons {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.control-buttons .el-button {
  width: 100%;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding-left: 20px;
}

.voice-control-btn {
  display: flex !important;
  align-items: center !important;
  justify-content: flex-start !important;
  padding-left: 20px !important;
}

.voice-control-btn .el-icon {
  margin-right: 8px !important;
  flex-shrink: 0;
}

.btn-text {
  text-align: left;
  display: inline-block;
}

.control-buttons .el-button .el-icon {
  margin-right: 5px;
}

.el-button {
  display: inline-flex;
  justify-content: center;
  align-items: center;
}

.el-button .el-icon {
  margin-right: 5px;
}

.el-button .el-icon + span {
  margin-left: 0;
}

.el-divider {
  margin: 16px 0;
}

.el-divider__text {
  font-size: 14px;
  color: var(--text-secondary);
}

.status-alert {
  margin-bottom: 16px;
}

.history-card {
  border: none;
  box-shadow: none;
}

.history-card :deep(.el-tabs__content) {
  padding: 20px 0;
}

.history-item {
  margin-bottom: 12px;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  transition: all 0.3s ease;
}

.history-item:hover {
  border-color: var(--primary-blue);
  transform: translateY(-1px);
}

.history-item-content {
  padding: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.history-text {
  flex: 1;
  margin-right: 12px;
  font-size: 14px;
  color: var(--text-primary);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* 右侧内容区布局 */
.right-content {
  flex: 1;
  display: flex;
  min-width: 0;
}

.transcription-area {
  flex: 1;
  padding: 10px;
  min-width: 0;
}

.player-area {
  width: 280px !important;
  padding: 20px 20px 20px 0;
  flex-shrink: 0;
}

.transcription-card {
  height: auto;
  min-height: 100%;
  display: flex;
  flex-direction: column;
  overflow: visible;
}

.transcription-card :deep(.el-card__body) {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: visible;
  padding-bottom: 15px;
}

.transcription-card :deep(.el-textarea) {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.transcription-card :deep(.el-textarea__inner) {
  height: 100% !important;
  flex: 1;
  padding: 10px;
  font-size: 13px;
  line-height: 1.4;
  color: var(--text-primary);
  background: var(--card-bg);
  border-color: var(--border-color);
  border-radius: 6px;
  transition: all 0.3s ease;
  min-height: 150px;
  max-height: 500px;
  resize: vertical;
  width: 100% !important;
}

.transcription-card :deep(.el-textarea__inner:focus) {
  border-color: var(--primary-blue);
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.1);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-header h3 {
  margin: 0;
  font-size: 16px;
  color: var(--card-title-color);
  transition: color 0.3s ease;
}

.button-group {
  display: flex;
  gap: 8px;
}

/* 音乐播放器样式 */
.music-player-card {
  height: 100%;
  text-align: center;
}

.music-player-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
}

.player-controls {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.cover-image {
  width: 160px;
  height: 160px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
  margin: 0 auto;
}

.cover-image:hover {
  transform: scale(1.02);
}

.song-info {
  margin: 0;
  font-size: 16px;
  font-weight: 500;
  color: var(--text-primary);
}

/* 主题按钮样式 */
.el-button--primary {
  background-color: var(--button-primary);
  border-color: var(--button-primary);
  transition: background-color 0.3s, border-color 0.3s;
}

.el-button--success {
  background-color: var(--button-success);
  border-color: var(--button-success);
  transition: background-color 0.3s, border-color 0.3s;
}

.el-button--danger {
  background-color: var(--button-danger);
  border-color: var(--button-danger);
  transition: background-color 0.3s, border-color 0.3s;
}

.el-button--info {
  background-color: var(--button-info);
  border-color: var(--button-info);
  transition: background-color 0.3s, border-color 0.3s;
}

/* 按钮悬浮效果 */
.el-button--primary:hover {
  background-color: var(--button-primary);
  border-color: var(--button-primary);
  opacity: 0.8;
}

.el-button--success:hover {
  background-color: var(--button-success);
  border-color: var(--button-success);
  opacity: 0.8;
}

.el-button--danger:hover {
  background-color: var(--button-danger);
  border-color: var(--button-danger);
  opacity: 0.8;
}

.el-button--info:hover {
  background-color: var(--button-info);
  border-color: var(--button-info);
  opacity: 0.8;
}

/* 深色模式特殊处理 */
body.dark {
  .el-card,
  .el-input__inner,
  .el-textarea__inner {
    background-color: var(--card-bg);
    border-color: #4c4c4c;
    color: var(--text-color);
  }

  .el-button:not(.el-button--primary):not(.el-button--success):not(.el-button--danger) {
    background-color: #4c4c4c;
    border-color: #4c4c4c;
    color: #ffffff;
  }

  .el-dropdown-menu {
    background-color: var(--card-bg);
    border-color: #4c4c4c;
  }

  .el-dropdown-menu__item {
    color: var(--text-color);
  }

  .el-dropdown-menu__item:hover {
    background-color: #3a3a3a;
  }

  .el-dropdown-item-group__title {
    color: #e0e0e0;
  }

  .history-item {
    border-color: #4c4c4c;
  }

  .el-tabs__nav-wrap::after {
    background-color: #4c4c4c;
  }

  .control-sidebar {
    border-color: #4c4c4c;
  }

  .el-alert {
    background-color: var(--card-bg);
    color: var(--text-color);
  }

  .el-alert--info .el-alert__title,
  .el-alert--success .el-alert__title {
    color: var(--text-color);
  }
}

/* 响应式布局 */
@media (max-width: 1200px) {
  .app-header {
    padding: 16px;
  }

  .control-sidebar {
    width: 100% !important;
    border-right: none;
    border-bottom: 1px solid var(--border-color);
  }

  .control-buttons {
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    gap: 10px;
  }

  .control-buttons .el-button {
    flex: 1 0 calc(50% - 5px);
    max-width: calc(50% - 5px);
  }

  .main-container {
    flex-direction: column;
  }

  .right-content {
    flex-direction: column;
  }

  .player-area {
    width: 100% !important;
    padding: 0 20px 20px 20px;
  }

  .function-buttons .el-button {
    flex: 1 0 calc(33.33% - 7px);
    max-width: calc(33.33% - 7px);
  }
}

@media (max-width: 768px) {
  .function-buttons .el-button {
    min-width: 100%;
  }
}

/* 自定义按钮样式 */
.custom-button {
  display: flex;
  align-items: center;
  padding: 8px 20px;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 14px;
}

.custom-button .icon {
  display: flex;
  align-items: center;
  margin-right: 8px;
  font-size: 16px;
}

.custom-button .text {
  text-align: left;
}

.primary-btn {
  background-color: var(--button-primary);
  color: white;
  transition: background-color 0.3s;
}

.primary-btn:hover {
  background-color: var(--button-primary);
  opacity: 0.85;
}

.danger-btn {
  background-color: var(--button-danger);
  color: white;
  transition: background-color 0.3s;
}

.danger-btn:hover {
  background-color: var(--button-danger);
  opacity: 0.85;
}

.default-btn {
  background-color: var(--card-bg);
  color: var(--text-color);
  border: 1px solid var(--border-color);
  transition: background-color 0.3s ease, color 0.3s ease, border-color 0.3s ease;
}

.default-btn:hover {
  background-color: var(--bg-color);
}

.custom-button.disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.control-buttons .el-button {
  width: 100%;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding-left: 20px;
}

.voice-control-btn {
  display: flex !important;
  align-items: center !important;
  justify-content: flex-start !important;
  padding-left: 20px !important;
}

.voice-control-btn .el-icon {
  margin-right: 8px !important;
  flex-shrink: 0;
}

.btn-text {
  text-align: left;
  display: inline-block;
}

.control-buttons .el-button .el-icon {
  margin-right: 5px;
}

/* 确保所有按钮内部的图标和文本对齐 */
.log-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-height: calc(100vh - 700px);
  overflow-y: auto;
}

.log-items {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.log-item {
  padding: 6px 12px;
  font-size: 13px;
}

.history-btn {
  padding: 8px 12px;
  background-color: #F5F7FA;
  color: var(--text-primary);
  border: 1px solid #DCDFE6;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.history-btn .text {
  flex: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-right: 10px;
}

.history-btn:hover {
  background-color: #ECEFF5;
  border-color: var(--primary-blue);
}

.restore-btn {
  margin-left: auto;
  flex-shrink: 0;
}

.history-card :deep(.el-tabs__header) {
  margin-bottom: 10px;
}

.history-card :deep(.el-tabs--border-card) {
  border: none;
  box-shadow: none;
}

.history-card :deep(.el-tabs--border-card > .el-tabs__header) {
  background-color: transparent;
  border-bottom: 1px solid var(--border-color);
}

.history-dropdown-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  padding-right: 5px;
}

.history-preview {
  max-width: 180px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-right: 10px;
}

.history-card :deep(.el-dropdown-menu__item.is-active) {
  color: var(--primary-blue);
  font-weight: bold;
}

.history-card :deep(.el-dropdown-menu__item.is-active:hover) {
  background-color: #ecf5ff;
}

.history-card :deep(.el-dropdown) {
  display: block;
  width: 100%;
}

/* 底栏样式 */
.app-footer {
  background-color: var(--footer-bg);
  color: var(--footer-text);
  padding: 12px 24px;
  width: 100%;
  margin-top: auto;
  transition: background-color 0.3s ease, color 0.3s ease;
}

.footer-content {
  display: flex;
  justify-content: center;
  align-items: center;
  max-width: 1200px;
  margin: 0 auto;
}

.footer-content p {
  margin: 0;
  font-size: 14px;
}

.help-command {
  font-weight: bold;
  color: var(--button-primary);
  background-color: rgba(64, 158, 255, 0.1);
  padding: 2px 5px;
  border-radius: 3px;
  transition: color 0.3s ease, background-color 0.3s ease;
}

body.dark .help-command {
  background-color: rgba(64, 158, 255, 0.2);
}

/* 添加警告按钮的夜间模式样式 */
body.light {
  --button-warning: #e6a23c;
}

body.dark {
  --button-warning: #8c6024;
}

.el-button--warning {
  background-color: var(--button-warning);
  border-color: var(--button-warning);
  transition: background-color 0.3s, border-color 0.3s;
}

.el-button--warning:hover {
  background-color: var(--button-warning);
  border-color: var(--button-warning);
  opacity: 0.8;
}

.transcription-card :deep(.el-card__header) {
  padding: 10px 15px;
}
</style>