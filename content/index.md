---
title: 我的知识库
tags:
  
---


<style>
  .card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 20px;
    margin-top: 20px;
    margin-bottom: 30px;
  }
  .nav-card {
    background-color: var(--light); 
    border: 1px solid var(--lightgray);
    border-radius: 12px !important; 
    padding: 12px !important;
    text-decoration: none !important;
    color: inherit;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    height: 100%;
    box-sizing: border-box;
	cursor: pointer; /* 强制显示手型光标 */
  }
  .nav-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(0,0,0,0.05);
    border-color: var(--secondary);
  }
  .card-title {
    font-size: 1.2rem;
    font-weight: bold;
    margin-bottom: 10px;
    color: var(--dark);
  }
  .card-desc {
    font-size: 0.9rem;
    color: var(--gray);
    margin-bottom: 20px;
    line-height: 1.5;
	flex-grow: 1; /* 让描述区域自动填充空白，保证按钮对齐 */
  }
  .card-btn {
    display: inline-block;
    padding: 8px 16px;
    background-color: var(--secondary);
    color: white !important; /* 强制文字颜色 */;
    border-radius: 60px;
    text-align: center;
    font-weight: 500;
    margin-top: auto;
	/* 注释 */
  }

  
</style>
 
<div class="card-grid">
  <a href="/Windows" class="nav-card">
    <div>
      <div class="card-title">Windows</div>
      <div class="card-desc">系统优化、注册表技巧、PowerShell 脚本及日常维护指南。</div>
    </div>
    <div class="card-btn">进入专区 &rarr;</div>
  </a>

  <a href="/macOS" class="nav-card">
    <div>
      <div class="card-title">macOS</div>
      <div class="card-desc">Mac 效率工具推荐、终端配置、Homebrew 使用及系统深度定制。</div>
    </div>
    <div class="card-btn">进入专区 &rarr;</div>
  </a>

  <a href="/Linux" class="nav-card">
    <div>
      <div class="card-title">Linux</div>
      <div class="card-desc">Ubuntu/CentOS 服务器运维、Docker 容器化部署、Shell 编程与内核调试。</div>
    </div>
    <div class="card-btn">进入专区 &rarr;</div>
  </a>

  <a href="/Network" class="nav-card">
    <div>
      <div class="card-title">网络专区</div>
      <div class="card-desc">TCP/IP 协议分析、路由交换配置、网络安全基础及抓包实战笔记。</div>
    </div>
    <div class="card-btn">进入专区 &rarr;</div>
  </a>

  <a href="/Study" class="nav-card">
    <div>
      <div class="card-title">学习专区</div>
      <div class="card-desc">Obsidian 使用心得、编程语言学习路线、读书笔记及个人思考复盘。</div>
    </div>
    <div class="card-btn">进入专区 &rarr;</div>
  </a>
  
  <a href="/Others" class="nav-card">
    <div>
      <div class="card-title">其他专区</div>
      <div class="card-desc">生活随笔、软件资源分享、未分类灵感及各类杂项记录。</div>
    </div>
    <div class="card-btn">进入专区 &rarr;</div>
  </a>
 
</div>
 

<div class="nav-card" style="position: relative; width: 100%; height: 333px; margin-top: 20px;">
    <iframe 
        src="/search-box.html" 
        style="width: 100%; height: 100%; border: none; overflow: hidden;" 
        scrolling="no">
    </iframe>
</div>


<div style="display: flex; justify-content: space-between; align-items: center; padding: 0px 0; font-size: 13px; color: #666; ">
    <div>
        &copy; 2017-2026 魔客室 mocos.cn | 用心分享技术
    </div> 
    <div style="display: flex; gap: 30px; align-items: center;">
        <a href="https://beian.miit.gov.cn/" target="_blank" style="color: #666; text-decoration: none;">
            渝ICP备17007546号-2
        </a>   
        <a href="http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=50022402000573" target="_blank" style="color: #666; text-decoration: none; display: flex; align-items: center; gap: 5px;">           
            <img src="assets/备案图标.png" alt="公网安备图标" style="width: 16px; height: 16px; vertical-align: middle;">
            <span>渝公网安备 50022402000573号</span>
        </a>
    </div>
</div>
<!-- 注释 -->

  