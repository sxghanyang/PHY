<!DOCTYPE html><html><head>
      <title>memory</title>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      
      <link rel="stylesheet" href="file:///c:\Users\user01\.vscode\extensions\shd101wyy.markdown-preview-enhanced-0.8.22\crossnote\dependencies\katex\katex.min.css">
      
      
      
      
      
      <style>
      code[class*=language-],pre[class*=language-]{color:#333;background:0 0;font-family:Consolas,"Liberation Mono",Menlo,Courier,monospace;text-align:left;white-space:pre;word-spacing:normal;word-break:normal;word-wrap:normal;line-height:1.4;-moz-tab-size:8;-o-tab-size:8;tab-size:8;-webkit-hyphens:none;-moz-hyphens:none;-ms-hyphens:none;hyphens:none}pre[class*=language-]{padding:.8em;overflow:auto;border-radius:3px;background:#f5f5f5}:not(pre)>code[class*=language-]{padding:.1em;border-radius:.3em;white-space:normal;background:#f5f5f5}.token.blockquote,.token.comment{color:#969896}.token.cdata{color:#183691}.token.doctype,.token.macro.property,.token.punctuation,.token.variable{color:#333}.token.builtin,.token.important,.token.keyword,.token.operator,.token.rule{color:#a71d5d}.token.attr-value,.token.regex,.token.string,.token.url{color:#183691}.token.atrule,.token.boolean,.token.code,.token.command,.token.constant,.token.entity,.token.number,.token.property,.token.symbol{color:#0086b3}.token.prolog,.token.selector,.token.tag{color:#63a35c}.token.attr-name,.token.class,.token.class-name,.token.function,.token.id,.token.namespace,.token.pseudo-class,.token.pseudo-element,.token.url-reference .token.variable{color:#795da3}.token.entity{cursor:help}.token.title,.token.title .token.punctuation{font-weight:700;color:#1d3e81}.token.list{color:#ed6a43}.token.inserted{background-color:#eaffea;color:#55a532}.token.deleted{background-color:#ffecec;color:#bd2c00}.token.bold{font-weight:700}.token.italic{font-style:italic}.language-json .token.property{color:#183691}.language-markup .token.tag .token.punctuation{color:#333}.language-css .token.function,code.language-css{color:#0086b3}.language-yaml .token.atrule{color:#63a35c}code.language-yaml{color:#183691}.language-ruby .token.function{color:#333}.language-markdown .token.url{color:#795da3}.language-makefile .token.symbol{color:#795da3}.language-makefile .token.variable{color:#183691}.language-makefile .token.builtin{color:#0086b3}.language-bash .token.keyword{color:#0086b3}pre[data-line]{position:relative;padding:1em 0 1em 3em}pre[data-line] .line-highlight-wrapper{position:absolute;top:0;left:0;background-color:transparent;display:block;width:100%}pre[data-line] .line-highlight{position:absolute;left:0;right:0;padding:inherit 0;margin-top:1em;background:hsla(24,20%,50%,.08);background:linear-gradient(to right,hsla(24,20%,50%,.1) 70%,hsla(24,20%,50%,0));pointer-events:none;line-height:inherit;white-space:pre}pre[data-line] .line-highlight:before,pre[data-line] .line-highlight[data-end]:after{content:attr(data-start);position:absolute;top:.4em;left:.6em;min-width:1em;padding:0 .5em;background-color:hsla(24,20%,50%,.4);color:#f4f1ef;font:bold 65%/1.5 sans-serif;text-align:center;vertical-align:.3em;border-radius:999px;text-shadow:none;box-shadow:0 1px #fff}pre[data-line] .line-highlight[data-end]:after{content:attr(data-end);top:auto;bottom:.4em}html body{font-family:'Helvetica Neue',Helvetica,'Segoe UI',Arial,freesans,sans-serif;font-size:16px;line-height:1.6;color:#333;background-color:#fff;overflow:initial;box-sizing:border-box;word-wrap:break-word}html body>:first-child{margin-top:0}html body h1,html body h2,html body h3,html body h4,html body h5,html body h6{line-height:1.2;margin-top:1em;margin-bottom:16px;color:#000}html body h1{font-size:2.25em;font-weight:300;padding-bottom:.3em}html body h2{font-size:1.75em;font-weight:400;padding-bottom:.3em}html body h3{font-size:1.5em;font-weight:500}html body h4{font-size:1.25em;font-weight:600}html body h5{font-size:1.1em;font-weight:600}html body h6{font-size:1em;font-weight:600}html body h1,html body h2,html body h3,html body h4,html body h5{font-weight:600}html body h5{font-size:1em}html body h6{color:#5c5c5c}html body strong{color:#000}html body del{color:#5c5c5c}html body a:not([href]){color:inherit;text-decoration:none}html body a{color:#08c;text-decoration:none}html body a:hover{color:#00a3f5;text-decoration:none}html body img{max-width:100%}html body>p{margin-top:0;margin-bottom:16px;word-wrap:break-word}html body>ol,html body>ul{margin-bottom:16px}html body ol,html body ul{padding-left:2em}html body ol.no-list,html body ul.no-list{padding:0;list-style-type:none}html body ol ol,html body ol ul,html body ul ol,html body ul ul{margin-top:0;margin-bottom:0}html body li{margin-bottom:0}html body li.task-list-item{list-style:none}html body li>p{margin-top:0;margin-bottom:0}html body .task-list-item-checkbox{margin:0 .2em .25em -1.8em;vertical-align:middle}html body .task-list-item-checkbox:hover{cursor:pointer}html body blockquote{margin:16px 0;font-size:inherit;padding:0 15px;color:#5c5c5c;background-color:#f0f0f0;border-left:4px solid #d6d6d6}html body blockquote>:first-child{margin-top:0}html body blockquote>:last-child{margin-bottom:0}html body hr{height:4px;margin:32px 0;background-color:#d6d6d6;border:0 none}html body table{margin:10px 0 15px 0;border-collapse:collapse;border-spacing:0;display:block;width:100%;overflow:auto;word-break:normal;word-break:keep-all}html body table th{font-weight:700;color:#000}html body table td,html body table th{border:1px solid #d6d6d6;padding:6px 13px}html body dl{padding:0}html body dl dt{padding:0;margin-top:16px;font-size:1em;font-style:italic;font-weight:700}html body dl dd{padding:0 16px;margin-bottom:16px}html body code{font-family:Menlo,Monaco,Consolas,'Courier New',monospace;font-size:.85em;color:#000;background-color:#f0f0f0;border-radius:3px;padding:.2em 0}html body code::after,html body code::before{letter-spacing:-.2em;content:'\00a0'}html body pre>code{padding:0;margin:0;word-break:normal;white-space:pre;background:0 0;border:0}html body .highlight{margin-bottom:16px}html body .highlight pre,html body pre{padding:1em;overflow:auto;line-height:1.45;border:#d6d6d6;border-radius:3px}html body .highlight pre{margin-bottom:0;word-break:normal}html body pre code,html body pre tt{display:inline;max-width:initial;padding:0;margin:0;overflow:initial;line-height:inherit;word-wrap:normal;background-color:transparent;border:0}html body pre code:after,html body pre code:before,html body pre tt:after,html body pre tt:before{content:normal}html body blockquote,html body dl,html body ol,html body p,html body pre,html body ul{margin-top:0;margin-bottom:16px}html body kbd{color:#000;border:1px solid #d6d6d6;border-bottom:2px solid #c7c7c7;padding:2px 4px;background-color:#f0f0f0;border-radius:3px}@media print{html body{background-color:#fff}html body h1,html body h2,html body h3,html body h4,html body h5,html body h6{color:#000;page-break-after:avoid}html body blockquote{color:#5c5c5c}html body pre{page-break-inside:avoid}html body table{display:table}html body img{display:block;max-width:100%;max-height:100%}html body code,html body pre{word-wrap:break-word;white-space:pre}}.markdown-preview{width:100%;height:100%;box-sizing:border-box}.markdown-preview ul{list-style:disc}.markdown-preview ul ul{list-style:circle}.markdown-preview ul ul ul{list-style:square}.markdown-preview ol{list-style:decimal}.markdown-preview ol ol,.markdown-preview ul ol{list-style-type:lower-roman}.markdown-preview ol ol ol,.markdown-preview ol ul ol,.markdown-preview ul ol ol,.markdown-preview ul ul ol{list-style-type:lower-alpha}.markdown-preview .newpage,.markdown-preview .pagebreak{page-break-before:always}.markdown-preview pre.line-numbers{position:relative;padding-left:3.8em;counter-reset:linenumber}.markdown-preview pre.line-numbers>code{position:relative}.markdown-preview pre.line-numbers .line-numbers-rows{position:absolute;pointer-events:none;top:1em;font-size:100%;left:0;width:3em;letter-spacing:-1px;border-right:1px solid #999;-webkit-user-select:none;-moz-user-select:none;-ms-user-select:none;user-select:none}.markdown-preview pre.line-numbers .line-numbers-rows>span{pointer-events:none;display:block;counter-increment:linenumber}.markdown-preview pre.line-numbers .line-numbers-rows>span:before{content:counter(linenumber);color:#999;display:block;padding-right:.8em;text-align:right}.markdown-preview .mathjax-exps .MathJax_Display{text-align:center!important}.markdown-preview:not([data-for=preview]) .code-chunk .code-chunk-btn-group{display:none}.markdown-preview:not([data-for=preview]) .code-chunk .status{display:none}.markdown-preview:not([data-for=preview]) .code-chunk .output-div{margin-bottom:16px}.markdown-preview .md-toc{padding:0}.markdown-preview .md-toc .md-toc-link-wrapper .md-toc-link{display:inline;padding:.25rem 0}.markdown-preview .md-toc .md-toc-link-wrapper .md-toc-link div,.markdown-preview .md-toc .md-toc-link-wrapper .md-toc-link p{display:inline}.markdown-preview .md-toc .md-toc-link-wrapper.highlighted .md-toc-link{font-weight:800}.scrollbar-style::-webkit-scrollbar{width:8px}.scrollbar-style::-webkit-scrollbar-track{border-radius:10px;background-color:transparent}.scrollbar-style::-webkit-scrollbar-thumb{border-radius:5px;background-color:rgba(150,150,150,.66);border:4px solid rgba(150,150,150,.66);background-clip:content-box}html body[for=html-export]:not([data-presentation-mode]){position:relative;width:100%;height:100%;top:0;left:0;margin:0;padding:0;overflow:auto}html body[for=html-export]:not([data-presentation-mode]) .markdown-preview{position:relative;top:0;min-height:100vh}@media screen and (min-width:914px){html body[for=html-export]:not([data-presentation-mode]) .markdown-preview{padding:2em calc(50% - 457px + 2em)}}@media screen and (max-width:914px){html body[for=html-export]:not([data-presentation-mode]) .markdown-preview{padding:2em}}@media screen and (max-width:450px){html body[for=html-export]:not([data-presentation-mode]) .markdown-preview{font-size:14px!important;padding:1em}}@media print{html body[for=html-export]:not([data-presentation-mode]) #sidebar-toc-btn{display:none}}html body[for=html-export]:not([data-presentation-mode]) #sidebar-toc-btn{position:fixed;bottom:8px;left:8px;font-size:28px;cursor:pointer;color:inherit;z-index:99;width:32px;text-align:center;opacity:.4}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] #sidebar-toc-btn{opacity:1}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc{position:fixed;top:0;left:0;width:300px;height:100%;padding:32px 0 48px 0;font-size:14px;box-shadow:0 0 4px rgba(150,150,150,.33);box-sizing:border-box;overflow:auto;background-color:inherit}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc::-webkit-scrollbar{width:8px}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc::-webkit-scrollbar-track{border-radius:10px;background-color:transparent}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc::-webkit-scrollbar-thumb{border-radius:5px;background-color:rgba(150,150,150,.66);border:4px solid rgba(150,150,150,.66);background-clip:content-box}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc a{text-decoration:none}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc .md-toc{padding:0 16px}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc .md-toc .md-toc-link-wrapper .md-toc-link{display:inline;padding:.25rem 0}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc .md-toc .md-toc-link-wrapper .md-toc-link div,html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc .md-toc .md-toc-link-wrapper .md-toc-link p{display:inline}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .md-sidebar-toc .md-toc .md-toc-link-wrapper.highlighted .md-toc-link{font-weight:800}html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .markdown-preview{left:300px;width:calc(100% - 300px);padding:2em calc(50% - 457px - 300px / 2);margin:0;box-sizing:border-box}@media screen and (max-width:1274px){html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .markdown-preview{padding:2em}}@media screen and (max-width:450px){html body[for=html-export]:not([data-presentation-mode])[html-show-sidebar-toc] .markdown-preview{width:100%}}html body[for=html-export]:not([data-presentation-mode]):not([html-show-sidebar-toc]) .markdown-preview{left:50%;transform:translateX(-50%)}html body[for=html-export]:not([data-presentation-mode]):not([html-show-sidebar-toc]) .md-sidebar-toc{display:none}
/* Please visit the URL below for more information: */
/*   https://shd101wyy.github.io/markdown-preview-enhanced/#/customize-css */

      </style>
      <script type="text/javascript">
  document.addEventListener("DOMContentLoaded", function () {
    // your code here
  });
</script>
    </head>
    <body for="html-export">
      <div class="crossnote markdown-preview  ">
      
<p><!-- filepath: d:\PHY-main\PHY-main\aerial-cuda-accelerated-ran-main\aerial-cuda-accelerated-ran-main\cuPHY\memory.md --></p>
<h1 id="cuphy-项目-cuda-加载逻辑与内存管理分析">cuPHY 项目 CUDA 加载逻辑与内存管理分析 </h1>
<h2 id="1-总览">1. 总览 </h2>
<p>本项目涉及多个模块，每个模块有不同的 CUDA 初始化和内存管理策略。整体分为以下几个层次：</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>┌──────────────────────────────────────────────────────────────┐
│                   内存管理分层架构                             │
├──────────────────────────────────────────────────────────────┤
│ 追踪层: memfoot_global.h (全局内存跟踪宏)                     │
├──────────────────────────────────────────────────────────────┤
│ 分配器层: hpinned_alloc / gpinned_buffer / IOBuf&lt;Alloc&gt;      │
├──────────────────────────────────────────────────────────────┤
│ CUDA 层: cudaMalloc / cudaHostAlloc / cuMemAlloc / GDRCopy   │
├──────────────────────────────────────────────────────────────┤
│ 硬件层: GPU Global Memory / Pinned Host / GDR / DOCA / DPDK  │
└──────────────────────────────────────────────────────────────┘
</code></pre><hr>
<h2 id="2-cuda-上下文与设备加载逻辑">2. CUDA 上下文与设备加载逻辑 </h2>
<h3 id="21-cuphy-核心库-srccuphy">2.1 cuPHY 核心库 (<code>src/cuphy/</code>) </h3>
<p><strong>初始化方式</strong>: 延迟初始化 (Lazy Init)</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>cuphy_context.cpp:
  cudaGetDevice() → 获取当前设备索引
  cudaDeviceGetAttribute() → 查询 compute capability、SM 数量、共享内存大小等
  → 创建内部 context 对象 (cuphy_i::context)
</code></pre><ul>
<li><strong>不主动创建 CUDA Context</strong> — 依赖调用者（上层 cuPHY-CP）已设置好 CUDA 上下文</li>
<li><strong>设备属性查询</strong>: <code>cudaDevAttrComputeCapabilityMajor/Minor</code>, <code>cudaDevAttrMaxSharedMemoryPerBlockOptin</code>, <code>cudaDevAttrMultiProcessorCount</code></li>
<li><strong>上下文封装</strong>: <code>cuphy::cudaContext</code> 和 <code>cuphy::cudaGreenContext</code> 提供 RAII 封装</li>
</ul>
<h3 id="22-cuphy-cp-cuphydriver-cuphydriver">2.2 cuPHY-CP cuphydriver (<code>cuphydriver/</code>) </h3>
<p><strong>初始化方式</strong>: 显式多上下文创建 + MPS/Green Context 分区</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>context.cpp (PhyDriverCtx 构造):
  1. 创建 MPS 上下文 (MpsCtx) — 每个信道类型一个
     ├── pdschMpsCtx  (下行 PDSCH)
     ├── pdcchMpsCtx  (下行 PDCCH)
     ├── puschMpsCtx  (上行 PUSCH)
     ├── pucchMpsCtx  (上行 PUCCH)
     ├── prachMpsCtx  (上行 PRACH)
     ├── srsMpsCtx    (SRS)
     ├── dlBfwMpsCtx  (下行波束赋形)
     ├── ulBfwMpsCtx  (上行波束赋形)
     └── gpuCommsMpsCtx (GPU 通信)

  2. 每个 MPS 上下文创建对应 CUDA Stream
     mpsCtx-&gt;setCtx()  // cuCtxSetCurrent()
     cudaStreamCreateWithPriority(&amp;stream, cudaStreamNonBlocking, priority)
     warmupStream(stream)  // 预热: 发射空 kernel + order kernel

  3. 创建 PHY 聚合对象 (PhyXxxAggr)
     new PhyPuschAggr / PhyPucchAggr / PhyPrachAggr / ...
</code></pre><p><strong>MPS 上下文管理</strong> (<code>mps.cpp</code>):</p>
<table>
<thead>
<tr>
<th>模式</th>
<th>实现</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>传统 MPS</strong></td>
<td><code>cuCtxCreate_v3()</code> + <code>CU_EXEC_AFFINITY_TYPE_SM_COUNT</code></td>
<td>限制每个上下文的 SM 数量</td>
</tr>
<tr>
<td><strong>Green Context</strong> (CUDA ≥ 12.4)</td>
<td><code>cuDevSmResourceSplitByCount()</code> + <code>cuGreenCtxCreate()</code></td>
<td>更细粒度的 GPU 资源分区</td>
</tr>
<tr>
<td><strong>Work Queue</strong> (CUDA ≥ 13.1)</td>
<td>Green Context + Work Queue</td>
<td>支持并发限制的工作队列</td>
</tr>
</tbody>
</table>
<p><strong>CUDA Stream 优先级分配</strong>:</p>
<table>
<thead>
<tr>
<th>Stream</th>
<th>优先级</th>
<th>用途</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>stream_order_pd</code> / <code>stream_order_srs_pd</code></td>
<td>-5</td>
<td>排序 kernel (最高优先级)</td>
</tr>
<tr>
<td><code>aggr_stream_pdsch</code></td>
<td>-4</td>
<td>PDSCH 下行</td>
</tr>
<tr>
<td><code>aggr_stream_csirs</code></td>
<td>-4</td>
<td>CSI-RS</td>
</tr>
<tr>
<td><code>aggr_stream_pucch</code></td>
<td>-3</td>
<td>PUCCH 上行</td>
</tr>
<tr>
<td><code>aggr_stream_prach</code></td>
<td>-3</td>
<td>PRACH</td>
</tr>
<tr>
<td><code>aggr_stream_pusch</code></td>
<td>-2</td>
<td>PUSCH 上行</td>
</tr>
<tr>
<td><code>aggr_stream_srs</code></td>
<td>-2</td>
<td>SRS</td>
</tr>
<tr>
<td><code>aggr_stream_ulbfw</code> / <code>aggr_stream_dlbfw</code></td>
<td>-2</td>
<td>波束赋形</td>
</tr>
<tr>
<td><code>stream_timing_dl</code> / <code>stream_timing_ul</code></td>
<td>-1</td>
<td>计时</td>
</tr>
</tbody>
</table>
<h3 id="23-cumac-cumac">2.3 cuMAC (<code>cuMAC/</code>) </h3>
<p><strong>初始化方式</strong>: 直接 <code>cudaMalloc</code> / <code>cudaFreeHost</code></p>
<ul>
<li>在 <code>cumacSubcontext</code> 构造/析构函数中大量直接调用 CUDA API</li>
<li>GPU 端结构体字段逐一分配: <code>cudaMalloc((void**)&amp;ptr, size)</code></li>
<li>CPU 端使用 pinned memory: <code>cudaMallocHost</code> / <code>cudaFreeHost</code></li>
<li>宏包装: <code>CUDA_CHECK_ERR()</code> 统一错误检查</li>
</ul>
<h3 id="24-nvipc-gt_common_libsnvipc">2.4 nvIPC (<code>gt_common_libs/nvIPC/</code>) </h3>
<p><strong>初始化方式</strong>: 多后端抽象</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>nv_ipc_cudapool.cu:
  cudaSetDevice(device_id)
  cudaStreamCreate(&amp;stream)        // 可选: CONFIG_CREATE_CUDA_STREAM
  cudaMalloc() / cuMemAlloc()      // 创建 GPU 内存池
</code></pre><h3 id="25-testbenches">2.5 testBenches </h3>
<p><strong>初始化方式</strong>: Green Context + Worker 线程</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>testWorker::createCuGreenCtx():
  my_green_context.bind()  // cuCtxSetCurrent()
  创建 cuphy::stream (RAII)
  创建 cuphy::event (RAII)
</code></pre><hr>
<h2 id="3-内存管理详细分析">3. 内存管理详细分析 </h2>
<h3 id="31-内存类型全景">3.1 内存类型全景 </h3>
<table>
<thead>
<tr>
<th>内存类型</th>
<th>分配 API</th>
<th>释放 API</th>
<th>使用场景</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>GPU Global Memory</strong></td>
<td><code>cudaMalloc</code> / <code>cuMemAlloc</code></td>
<td><code>cudaFree</code> / <code>cuMemFree</code></td>
<td>信号处理数据、信道参数</td>
</tr>
<tr>
<td><strong>Host Pinned Memory</strong></td>
<td><code>cudaHostAlloc</code> / <code>cudaMallocHost</code></td>
<td><code>cudaFreeHost</code></td>
<td>DMA 传输缓冲区</td>
</tr>
<tr>
<td><strong>GPU Constant Memory</strong></td>
<td><code>cudaMemcpyToSymbolAsync</code></td>
<td>—</td>
<td>PUCCH 常数表 (W1-W4, phi)</td>
</tr>
<tr>
<td><strong>GDR Pinned Buffer</strong></td>
<td><code>cuMemAlloc</code> + <code>gdr_pin_buffer</code> + <code>gdr_map</code></td>
<td><code>gdr_unmap</code> + <code>gdr_unpin</code> + <code>cuMemFree</code></td>
<td>前传零拷贝 RDMA</td>
</tr>
<tr>
<td><strong>DOCA GPU Memory</strong></td>
<td><code>doca_gpu_mem_alloc</code></td>
<td><code>doca_gpu_mem_free</code></td>
<td>BlueField DPU 加速</td>
</tr>
<tr>
<td><strong>DPDK Hugepage</strong></td>
<td><code>rte_gpu_mem_alloc</code></td>
<td><code>rte_gpu_mem_free</code></td>
<td>DPDK 网络传输</td>
</tr>
</tbody>
</table>
<h3 id="32-cuphy-核心库内存管理">3.2 cuPHY 核心库内存管理 </h3>
<h4 id="321-tensor-描述符-tensor_desccpphpp">3.2.1 Tensor 描述符 (<code>tensor_desc.cpp/.hpp</code>) </h4>
<ul>
<li>统一的多维数据描述: shape, dtype, strides, layout</li>
<li>不负责分配/释放内存，仅描述已分配内存的结构</li>
<li>支持 <code>cuphyTensorDescriptor</code> 的创建/复制/销毁</li>
</ul>
<h4 id="322-智能指针封装-examplescommoncuphyhpp">3.2.2 智能指针封装 (<code>examples/common/cuphy.hpp</code>) </h4>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// GPU 设备内存 RAII</span>
<span class="token keyword keyword-template">template</span><span class="token operator">&lt;</span><span class="token keyword keyword-class">class</span> <span class="token class-name">T</span><span class="token operator">&gt;</span> <span class="token keyword keyword-struct">struct</span> <span class="token class-name">device_deleter</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-void">void</span> <span class="token keyword keyword-operator">operator</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">(</span>T<span class="token operator">*</span> p<span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token function">cudaFree</span><span class="token punctuation">(</span>p<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
<span class="token keyword keyword-template">template</span><span class="token operator">&lt;</span><span class="token keyword keyword-typename">typename</span> <span class="token class-name">T</span><span class="token operator">&gt;</span>
<span class="token keyword keyword-using">using</span> unique_device_ptr <span class="token operator">=</span> std<span class="token double-colon punctuation">::</span>unique_ptr<span class="token operator">&lt;</span>T<span class="token punctuation">,</span> device_deleter<span class="token operator">&lt;</span>T<span class="token operator">&gt;&gt;</span><span class="token punctuation">;</span>

<span class="token comment">// 带内存跟踪的分配</span>
<span class="token keyword keyword-template">template</span><span class="token operator">&lt;</span><span class="token keyword keyword-typename">typename</span> <span class="token class-name">T</span><span class="token operator">&gt;</span>
unique_device_ptr<span class="token operator">&lt;</span>T<span class="token operator">&gt;</span> <span class="token function">make_unique_device</span><span class="token punctuation">(</span>size_t count<span class="token punctuation">,</span> cuphyMemoryFootprint<span class="token operator">*</span> pMF<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    T<span class="token operator">*</span> p<span class="token punctuation">;</span> <span class="token function">cudaMalloc</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>p<span class="token punctuation">,</span> count <span class="token operator">*</span> <span class="token keyword keyword-sizeof">sizeof</span><span class="token punctuation">(</span>T<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token comment">// 可选: pMF 跟踪分配</span>
    <span class="token keyword keyword-return">return</span> <span class="token generic-function"><span class="token function">unique_device_ptr</span><span class="token generic class-name"><span class="token operator">&lt;</span>T<span class="token operator">&gt;</span></span></span><span class="token punctuation">(</span>p<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>

<span class="token comment">// Host Pinned 内存 RAII</span>
<span class="token keyword keyword-template">template</span><span class="token operator">&lt;</span><span class="token keyword keyword-class">class</span> <span class="token class-name">T</span><span class="token operator">&gt;</span> <span class="token keyword keyword-struct">struct</span> <span class="token class-name">pinned_deleter</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-void">void</span> <span class="token keyword keyword-operator">operator</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">(</span>T<span class="token operator">*</span> p<span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token function">cudaFreeHost</span><span class="token punctuation">(</span>p<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
<span class="token keyword keyword-template">template</span><span class="token operator">&lt;</span><span class="token keyword keyword-typename">typename</span> <span class="token class-name">T</span><span class="token operator">&gt;</span>
<span class="token keyword keyword-using">using</span> unique_pinned_ptr <span class="token operator">=</span> std<span class="token double-colon punctuation">::</span>unique_ptr<span class="token operator">&lt;</span>T<span class="token punctuation">,</span> pinned_deleter<span class="token operator">&lt;</span>T<span class="token operator">&gt;&gt;</span><span class="token punctuation">;</span>
</code></pre><h4 id="323-cuda-streamevent-raii-cuphystream-cuphyevent">3.2.3 CUDA Stream/Event RAII (<code>cuphy::stream</code>, <code>cuphy::event</code>) </h4>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token keyword keyword-class">class</span> <span class="token class-name">stream</span> <span class="token keyword keyword-final">final</span> <span class="token punctuation">{</span>
    cudaStream_t stream_<span class="token punctuation">;</span>
<span class="token keyword keyword-public">public</span><span class="token operator">:</span>
    <span class="token keyword keyword-explicit">explicit</span> <span class="token function">stream</span><span class="token punctuation">(</span><span class="token keyword keyword-unsigned">unsigned</span> <span class="token keyword keyword-int">int</span> flags<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token function">cudaStreamCreateWithFlags</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>stream_<span class="token punctuation">,</span> flags<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    <span class="token operator">~</span><span class="token function">stream</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token function">cudaStreamDestroy</span><span class="token punctuation">(</span>stream_<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>
    <span class="token comment">// move-only, 不可复制</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre><h3 id="33-cuphydriver-内存管理">3.3 cuphydriver 内存管理 </h3>
<h4 id="331-hpinned_alloc--host-pinned-分配器">3.3.1 <code>hpinned_alloc</code> — Host Pinned 分配器 </h4>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token keyword keyword-struct">struct</span> <span class="token class-name">hpinned_alloc</span> <span class="token punctuation">{</span>
    <span class="token keyword keyword-static">static</span> <span class="token keyword keyword-void">void</span><span class="token operator">*</span> <span class="token function">allocate</span><span class="token punctuation">(</span>size_t nbytes<span class="token punctuation">)</span> <span class="token punctuation">{</span>
        <span class="token keyword keyword-void">void</span><span class="token operator">*</span> addr<span class="token punctuation">;</span>
        <span class="token function">cudaHostAlloc</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>addr<span class="token punctuation">,</span> nbytes<span class="token punctuation">,</span> cudaHostAllocDefault <span class="token operator">|</span> cudaHostAllocPortable<span class="token punctuation">)</span><span class="token punctuation">;</span>
        <span class="token keyword keyword-return">return</span> addr<span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
    <span class="token keyword keyword-static">static</span> <span class="token keyword keyword-void">void</span> <span class="token function">deallocate</span><span class="token punctuation">(</span><span class="token keyword keyword-void">void</span><span class="token operator">*</span> addr<span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token function">cudaFreeHost</span><span class="token punctuation">(</span>addr<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>
    <span class="token keyword keyword-static">static</span> <span class="token keyword keyword-void">void</span> <span class="token function">clear</span><span class="token punctuation">(</span><span class="token keyword keyword-void">void</span><span class="token operator">*</span> addr<span class="token punctuation">,</span> size_t n<span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token function">memset</span><span class="token punctuation">(</span>addr<span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">,</span> n<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre><ul>
<li><strong><code>cudaHostAllocPortable</code></strong>: 使内存跨所有 CUDA 上下文可见</li>
<li>用于 <code>IOBuf&lt;hpinned_alloc&gt;</code> 模板化缓冲区</li>
</ul>
<h4 id="332-gpinned_buffer--gpudirect-rdma-缓冲区">3.3.2 <code>gpinned_buffer</code> — GPUDirect RDMA 缓冲区 </h4>
<pre data-role="codeBlock" data-info="" class="language-text"><code>分配流程:
  cuMemAlloc()                    // GPU 显存分配
  → GPU page alignment           // GPU 页对齐
  → cuPointerSetAttribute(SYNC_MEMOPS)  // 设置同步属性
  → gdr_pin_buffer()             // GDR pin (BAR1 映射)
  → gdr_map()                    // 映射到用户空间
  → 同时获得 addr_d (GPU) 和 addr_h (CPU) 地址

释放流程:
  gdr_unmap() → gdr_unpin_buffer() → cuMemFree()

回退方案 (无 RDMA):
  cudaHostAlloc() + cuMemHostGetDevicePointer()  // Unified Memory
</code></pre><ul>
<li><strong>用途</strong>: 前传 (Fronthaul) 驱动的零拷贝数据传输</li>
<li><strong>CPU 和 GPU 同时可访问</strong>: 通过 GDRCopy 实现 BAR1 映射</li>
</ul>
<h4 id="333-dlul-buffer-dlbuffercpp-ulbuffercpp">3.3.3 DL/UL Buffer (<code>dlbuffer.cpp</code>, <code>ulbuffer.cpp</code>) </h4>
<ul>
<li><strong>预分配</strong>: 在小区配置时一次性预分配上下行缓冲区</li>
<li><strong>循环复用</strong>: 通过 slot 索引循环使用缓冲区，避免运行时分配</li>
</ul>
<h4 id="334-harq-pool-harq_poolcpp">3.3.4 HARQ Pool (<code>harq_pool.cpp</code>) </h4>
<ul>
<li><strong>GPU 端预分配</strong>: 所有 UE 的 HARQ 软比特缓存一次性分配</li>
<li><strong>Pool 管理</strong>: 基于 UE ID 和 HARQ 进程号索引</li>
<li><strong>增量冗余</strong>: 支持软合并，缓存跨 slot 保持</li>
</ul>
<h4 id="335-pusch-聚合对象的内存-phypusch_aggrcpp">3.3.5 PUSCH 聚合对象的内存 (<code>phypusch_aggr.cpp</code>) </h4>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// Pinned host memory with GPU visibility</span>
<span class="token function">cudaHostAlloc</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>pFoCompensationBuffersInOut<span class="token punctuation">,</span> size<span class="token punctuation">,</span>
              cudaHostAllocPortable <span class="token operator">|</span> cudaHostAllocMapped<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">cudaHostAlloc</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>pPreEarlyHarqWaitKernelStatus<span class="token punctuation">,</span> <span class="token keyword keyword-sizeof">sizeof</span><span class="token punctuation">(</span><span class="token keyword keyword-uint8_t">uint8_t</span><span class="token punctuation">)</span><span class="token punctuation">,</span>
              cudaHostAllocPortable <span class="token operator">|</span> cudaHostAllocMapped<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><ul>
<li><strong><code>cudaHostAllocMapped</code></strong>: 允许 GPU 通过 Unified Address 直接访问 host 内存</li>
<li>用于 CPU-GPU 同步标志 (早期 HARQ 检测结果)</li>
</ul>
<h3 id="34-cumac-内存管理">3.4 cuMAC 内存管理 </h3>
<p><strong>模式</strong>: GPU/CPU 镜像结构体</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>cellGrpPrmsGpu  ←→  cellGrpPrmsCpu      // 参数结构体
schdSolGpu      ←→  schdSolCpu          // 调度结果
cellGrpUeStatusGpu ←→ cellGrpUeStatusCpu // UE 状态
</code></pre><ul>
<li>GPU 端: <code>cudaMalloc</code> 逐字段分配</li>
<li>CPU 端: <code>cudaMallocHost</code> (pinned) 或 <code>new</code> (pageable)</li>
<li>数据传输: <code>cudaMemcpy</code> / <code>cudaMemcpyAsync</code> + <code>cudaStreamSynchronize</code></li>
</ul>
<p><strong>析构</strong>: <code>cumacSubcontext::~cumacSubcontext()</code> 中逐字段释放，先检查非空再释放</p>
<h3 id="35-nvipc-内存管理">3.5 nvIPC 内存管理 </h3>
<p><strong>CUDA Pool</strong> (<code>nv_ipc_cudapool.cu</code>):</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>创建:
  cudaSetDevice(device_id)
  cudaMalloc() 分配大块 GPU 内存池
  可选创建专用 cudaStream

数据传输:
  带 stream: cudaMemcpyAsync + cudaStreamSynchronize
  无 stream: cudaMemcpy (同步)

关闭:
  cudaStreamDestroy()
  释放 GPU 内存池
</code></pre><p><strong>CUDA 工具</strong> (<code>nv_ipc_cuda_utils.cu</code>):</p>
<table>
<thead>
<tr>
<th>函数</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>nv_ipc_memcpy_to_host()</code></td>
<td><code>cudaMemcpy(DeviceToHost)</code></td>
</tr>
<tr>
<td><code>nv_ipc_memcpy_to_device()</code></td>
<td><code>cudaMemcpy(HostToDevice)</code></td>
</tr>
<tr>
<td><code>nv_ipc_cuda_register_host()</code></td>
<td><code>cudaHostRegister()</code> 注册已有 host 内存为 pinned</td>
</tr>
<tr>
<td><code>nv_ipc_cuda_unregister_host()</code></td>
<td><code>cudaHostUnregister()</code></td>
</tr>
</tbody>
</table>
<hr>
<h2 id="4-全局内存跟踪框架-memfoot_globalh">4. 全局内存跟踪框架 (<code>memfoot_global.h</code>) </h2>
<h3 id="41-设计">4.1 设计 </h3>
<p>项目实现了一套<strong>编译时可选</strong>的全局内存跟踪系统，通过宏替换标准 CUDA 分配/释放 API。</p>
<h3 id="42-跟踪分类">4.2 跟踪分类 </h3>
<table>
<thead>
<tr>
<th>标签</th>
<th>模块</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>MF_TAG_CUPHY_OTHER</code></td>
<td>cuPHY</td>
<td>cuPHY 库内部分配</td>
</tr>
<tr>
<td><code>MF_TAG_PHYDRV_OTHER</code></td>
<td>cuphydriver</td>
<td>PHY 驱动分配</td>
</tr>
<tr>
<td><code>MF_TAG_FH_DOCA_RX/TX</code></td>
<td>aerial-fh-driver</td>
<td>前传 DOCA 收发</td>
</tr>
<tr>
<td><code>MF_TAG_FH_GPU_COMM</code></td>
<td>aerial-fh-driver</td>
<td>GPU 通信</td>
</tr>
<tr>
<td><code>MF_TAG_FH_PEER</code></td>
<td>aerial-fh-driver</td>
<td>Peer 内存</td>
</tr>
</tbody>
</table>
<h3 id="43-跟踪宏示例">4.3 跟踪宏示例 </h3>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// 替代 cudaMalloc</span>
<span class="token function">MF_CUDA_MALLOC</span><span class="token punctuation">(</span>ptr<span class="token punctuation">,</span> size<span class="token punctuation">)</span>
→ <span class="token function">memfoot_global_get_mem_info</span><span class="token punctuation">(</span><span class="token punctuation">)</span>    <span class="token comment">// 记录分配前状态</span>
→ <span class="token function">cudaMalloc</span><span class="token punctuation">(</span>ptr<span class="token punctuation">,</span> size<span class="token punctuation">)</span>            <span class="token comment">// 实际分配</span>
→ <span class="token function">memfoot_global_gpu_size_add</span><span class="token punctuation">(</span><span class="token punctuation">)</span>    <span class="token comment">// 记录分配后 delta</span>

<span class="token comment">// 替代 cudaFree</span>
<span class="token function">MF_CUDA_FREE</span><span class="token punctuation">(</span>ptr<span class="token punctuation">)</span>
→ <span class="token function">memfoot_global_get_mem_info</span><span class="token punctuation">(</span><span class="token punctuation">)</span>    <span class="token comment">// 记录释放前状态</span>
→ <span class="token function">cudaFree</span><span class="token punctuation">(</span>ptr<span class="token punctuation">)</span>                    <span class="token comment">// 实际释放</span>
→ <span class="token function">memfoot_global_gpu_size_sub</span><span class="token punctuation">(</span><span class="token punctuation">)</span>    <span class="token comment">// 记录释放后 delta</span>

<span class="token comment">// 检测未跟踪的分配</span>
<span class="token function">MF_GPU_MEMFOOT_CHECK</span><span class="token punctuation">(</span><span class="token punctuation">)</span>             <span class="token comment">// 对比 cudaMemGetInfo 和跟踪计数</span>
</code></pre><h3 id="44-覆盖的-api">4.4 覆盖的 API </h3>
<table>
<thead>
<tr>
<th>类别</th>
<th>跟踪宏</th>
</tr>
</thead>
<tbody>
<tr>
<td>GPU 分配</td>
<td><code>MF_CUDA_MALLOC</code>, <code>MF_CU_MEM_ALLOC</code>, <code>MF_CUDA_MALLOC_ASYNC</code>, <code>MF_CUDA_MALLOC_FROM_POOL_ASYNC</code>, <code>MF_CUDA_MALLOC_3D</code>, <code>MF_CUDA_MALLOC_PITCH</code></td>
</tr>
<tr>
<td>GPU 释放</td>
<td><code>MF_CUDA_FREE</code>, <code>MF_CU_MEM_FREE</code>, <code>MF_CUDA_FREE_ASYNC</code></td>
</tr>
<tr>
<td>Host 分配</td>
<td><code>MF_CUDA_HOST_ALLOC</code>, <code>MF_CUDA_MALLOC_HOST</code>, <code>MF_CU_MEM_ALLOC_HOST</code></td>
</tr>
<tr>
<td>Host 释放</td>
<td><code>MF_CUDA_FREE_HOST</code>, <code>MF_CU_MEM_FREE_HOST</code></td>
</tr>
<tr>
<td>DOCA</td>
<td><code>MF_DOCA_GPU_MEM_ALLOC</code>, <code>MF_DOCA_GPU_MEM_FREE</code></td>
</tr>
<tr>
<td>DPDK</td>
<td><code>MF_RTE_GPU_MEM_ALLOC</code>, <code>MF_RTE_GPU_MEM_FREE</code></td>
</tr>
<tr>
<td>隐式分配</td>
<td><code>MF_CUDA_STREAM_CREATE_WITH_FLAGS</code>, <code>MF_CU_CTX_CREATE</code>, <code>MF_CU_GRAPH_INSTANTIATE</code></td>
</tr>
</tbody>
</table>
<hr>
<h2 id="5-各模块数据传输模式">5. 各模块数据传输模式 </h2>
<h3 id="51-cuphy-核心--cuda-constant-memory">5.1 cuPHY 核心 — CUDA Constant Memory </h3>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// PUCCH F3 前端: 将常数表加载到 GPU constant memory</span>
<span class="token function">cudaMemcpyToSymbolAsync</span><span class="token punctuation">(</span>d_W1<span class="token punctuation">,</span> W1<span class="token punctuation">,</span> <span class="token keyword keyword-sizeof">sizeof</span><span class="token punctuation">(</span>W1<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">,</span> cudaMemcpyHostToDevice<span class="token punctuation">,</span> strm<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">cudaMemcpyToSymbolAsync</span><span class="token punctuation">(</span>d_phi_6<span class="token punctuation">,</span> phi_6<span class="token punctuation">,</span> <span class="token keyword keyword-sizeof">sizeof</span><span class="token punctuation">(</span>d_phi_6<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><h3 id="52-cuphydriver--异步流传输">5.2 cuphydriver — 异步流传输 </h3>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// 上下文创建时: 预热 stream (避免首次 kernel launch 延迟)</span>
<span class="token function">warmupStream</span><span class="token punctuation">(</span>stream<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">launch_kernel_warmup</span><span class="token punctuation">(</span>stream<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token function">launch_kernel_order</span><span class="token punctuation">(</span>stream<span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    gpu_device<span class="token operator">-&gt;</span><span class="token function">synchronizeStream</span><span class="token punctuation">(</span>stream<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre><h3 id="53-cumac--批量-h2dd2h">5.3 cuMAC — 批量 H2D/D2H </h3>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// 典型模式: 批量上传参数到 GPU</span>
<span class="token function">cudaMalloc</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>gpu<span class="token operator">-&gt;</span>field<span class="token punctuation">,</span> size<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">cudaMemcpy</span><span class="token punctuation">(</span>gpu<span class="token operator">-&gt;</span>field<span class="token punctuation">,</span> host_data<span class="token punctuation">,</span> size<span class="token punctuation">,</span> cudaMemcpyHostToDevice<span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// 异步版本</span>
<span class="token function">cudaMemcpyAsync</span><span class="token punctuation">(</span>gpu<span class="token operator">-&gt;</span>field<span class="token punctuation">,</span> host_data<span class="token punctuation">,</span> size<span class="token punctuation">,</span> cudaMemcpyHostToDevice<span class="token punctuation">,</span> stream<span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre><h3 id="54-nvipc--零拷贝-ipc">5.4 nvIPC — 零拷贝 IPC </h3>
<pre data-role="codeBlock" data-info="" class="language-text"><code>SHM 模式:   POSIX 共享内存 + cudaHostRegister()
CUDA 模式:  cudaMalloc + cudaIpcGetMemHandle / cudaIpcOpenMemHandle
DPDK 模式:  rte_malloc hugepage 内存池
DOCA 模式:  DPU 上的 doca_gpu_mem_alloc
</code></pre><hr>
<h2 id="6-关键设计模式总结">6. 关键设计模式总结 </h2>
<h3 id="61-raii-resource-acquisition-is-initialization">6.1 RAII (Resource Acquisition Is Initialization) </h3>
<table>
<thead>
<tr>
<th>类</th>
<th>管理的资源</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>cuphy::stream</code></td>
<td><code>cudaStream_t</code></td>
</tr>
<tr>
<td><code>cuphy::event</code></td>
<td><code>cudaEvent_t</code></td>
</tr>
<tr>
<td><code>cuphy::cudaContext</code></td>
<td><code>CUcontext</code></td>
</tr>
<tr>
<td><code>cuphy::cudaGreenContext</code></td>
<td><code>CUgreenCtx</code></td>
</tr>
<tr>
<td><code>unique_device_ptr&lt;T&gt;</code></td>
<td>GPU 内存</td>
</tr>
<tr>
<td><code>unique_pinned_ptr&lt;T&gt;</code></td>
<td>Pinned Host 内存</td>
</tr>
<tr>
<td><code>gpinned_buffer</code></td>
<td>GDR 映射的 GPU/Host 内存对</td>
</tr>
<tr>
<td><code>MpsCtx</code></td>
<td>MPS/Green Context + 关联 SM 资源</td>
</tr>
</tbody>
</table>
<h3 id="62-预分配--池化">6.2 预分配 + 池化 </h3>
<ul>
<li><strong>HARQ Pool</strong>: 启动时预分配所有 UE 的软比特缓存</li>
<li><strong>DL/UL Buffer</strong>: 按最大 slot 数预分配循环缓冲区</li>
<li><strong>nvIPC CUDA Pool</strong>: 预分配大块 GPU 内存做 IPC 池</li>
<li><strong>聚合对象</strong>: 每个信道类型预创建 N 个 <code>PhyXxxAggr</code> 实例循环使用</li>
</ul>
<h3 id="63-多级-gpu-资源隔离">6.3 多级 GPU 资源隔离 </h3>
<pre data-role="codeBlock" data-info="" class="language-text"><code>GPU 设备
  └── MPS Context (SM 分区)
       ├── DL Context (PDSCH + PDCCH + PBCH + CSI-RS)
       ├── UL Context (PUSCH + PUCCH + PRACH)
       ├── SRS Context
       ├── BFW Context (UL + DL 波束赋形)
       └── GPU Comms Context
  每个 Context 内:
       └── 带优先级的 CUDA Streams (-5 ~ -1)
            └── 聚合对象 (PhyXxxAggr) 绑定到特定 stream
</code></pre><h3 id="64-零拷贝数据路径">6.4 零拷贝数据路径 </h3>
<pre data-role="codeBlock" data-info="" class="language-text"><code>前传 RU 数据 → [RDMA/DOCA] → gpinned_buffer (GPU+CPU 共享)
                                    ↓ GPU 直接处理
                              CUDA kernel (信道处理)
                                    ↓
                              结果 buffer → [nvIPC] → MAC 层
</code></pre><hr>
<h2 id="7-环境变量控制">7. 环境变量控制 </h2>
<table>
<thead>
<tr>
<th>变量</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>AERIAL_MEMTRACE</code></td>
<td>设为 <code>1</code> 启用全局内存跟踪 + kernel 预加载</td>
</tr>
</tbody>
</table>
<hr>
<h2 id="8-各模块对比总结">8. 各模块对比总结 </h2>
<table>
<thead>
<tr>
<th>模块</th>
<th>CUDA 初始化</th>
<th>内存分配风格</th>
<th>释放方式</th>
<th>特殊技术</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>cuPHY 核心</strong></td>
<td>延迟初始化, 依赖调用者上下文</td>
<td>RAII 智能指针 + Tensor 描述符</td>
<td>析构函数自动释放</td>
<td>Constant Memory, CUDA Graph</td>
</tr>
<tr>
<td><strong>cuphydriver</strong></td>
<td>显式多 MPS/Green Context</td>
<td>预分配 + <code>hpinned_alloc</code> + <code>gpinned_buffer</code></td>
<td>RAII + 显式销毁</td>
<td>GDR 零拷贝, MPS SM 分区, Stream 优先级</td>
</tr>
<tr>
<td><strong>cuMAC</strong></td>
<td>简单 <code>cudaSetDevice</code></td>
<td>逐字段 <code>cudaMalloc</code>/<code>cudaMallocHost</code></td>
<td>析构函数逐字段 <code>cudaFree</code></td>
<td>GPU-CPU 镜像结构体</td>
</tr>
<tr>
<td><strong>nvIPC</strong></td>
<td><code>cudaSetDevice</code> + 可选 stream</td>
<td>大块池分配</td>
<td>pool close 统一释放</td>
<td>多后端 (SHM/CUDA/DPDK/DOCA/UDP)</td>
</tr>
<tr>
<td><strong>testBenches</strong></td>
<td>Green Context bind</td>
<td>RAII (<code>unique_ptr</code>)</td>
<td>自动析构</td>
<td>Green Context 资源分区测试</td>
</tr>
<tr>
<td><strong>aerial-fh-driver</strong></td>
<td>DOCA/DPDK 初始化</td>
<td><code>gpinned_buffer</code> / DOCA</td>
<td>GDR unmap + unpin</td>
<td>GPUDirect RDMA 前传</td>
</tr>
</tbody>
</table>
<hr>
<h2 id="9-gpu-与-cpu-内存用量分析">9. GPU 与 CPU 内存用量分析 </h2>
<h3 id="91-内存类型总览">9.1 内存类型总览 </h3>
<table>
<thead>
<tr>
<th>内存类型</th>
<th>分配 API</th>
<th>访问方</th>
<th>典型用途</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>GPU Global</strong></td>
<td><code>cudaMalloc</code> / <code>cuMemAlloc</code></td>
<td>GPU only</td>
<td>信号处理、信道参数、调度矩阵</td>
</tr>
<tr>
<td><strong>Host Pinned</strong></td>
<td><code>cudaHostAlloc</code> / <code>cudaMallocHost</code></td>
<td>CPU (DMA-able)</td>
<td>H2D/D2H DMA 传输缓冲区</td>
</tr>
<tr>
<td><strong>Host Pinned + Mapped</strong></td>
<td><code>cudaHostAlloc</code> + <code>cudaHostAllocMapped</code></td>
<td>CPU + GPU (UA)</td>
<td>CPU-GPU 同步标志位</td>
</tr>
<tr>
<td><strong>GDR Mapped</strong></td>
<td><code>cuMemAlloc</code> + <code>gdr_pin_buffer</code> + <code>gdr_map</code></td>
<td>GPU + CPU (BAR1)</td>
<td>前传零拷贝 RDMA</td>
</tr>
<tr>
<td><strong>GPU Constant</strong></td>
<td><code>cudaMemcpyToSymbolAsync</code></td>
<td>GPU const cache</td>
<td>PUCCH 查找表 (W1W4, phi)</td>
</tr>
<tr>
<td><strong>DOCA GPU</strong></td>
<td><code>doca_gpu_mem_alloc</code></td>
<td>BlueField DPU</td>
<td>DPU 加速报文处理</td>
</tr>
<tr>
<td><strong>DPDK Hugepage</strong></td>
<td><code>rte_gpu_mem_alloc</code></td>
<td>GPU + DPDK</td>
<td>DPDK 网络传输缓冲区</td>
</tr>
</tbody>
</table>
<hr>
<h3 id="92-各模块-gpu-内存用量">9.2 各模块 GPU 内存用量 </h3>
<h4 id="921-cuphydriver--最大-gpu-内存消耗方">9.2.1 cuphydriver  最大 GPU 内存消耗方 </h4>
<p><strong>HARQ 软比特缓冲池</strong> (<code>harq_pool.hpp</code>):</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>HARQ_POOL_SIZE[] = {262144, 4000000, 8000000};  // bytes
// 3 个固定大小池，每池预分配 maxHarqPools 个缓冲区
Pool[0]: 256 KiB/buffer  x N    小 TB 软比特
Pool[1]: 3.8 MiB/buffer  x N    中 TB 软比特
Pool[2]: 7.6 MiB/buffer  x N    大 TB 软比特
总量: N x (256K + 3.8M + 7.6M) B GPU 内存
跟踪: HarqPool::gpuMemSize -&gt; mf.addGpuRegularSize()
</code></pre><p><strong>UL 输入缓冲区</strong> (<code>ulbuffer.cpp</code>, per cell x per slot):</p>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// 每个 ULInputBuffer 同时分配 GPU + CPU Pinned 各一份</span>
addr_d <span class="token operator">=</span> <span class="token keyword keyword-new">new</span> <span class="token function">dev_buf</span><span class="token punctuation">(</span>addr_sz<span class="token punctuation">)</span><span class="token punctuation">;</span>    <span class="token comment">// GPU: cudaMalloc</span>
addr_h <span class="token operator">=</span> <span class="token keyword keyword-new">new</span> <span class="token function">host_buf</span><span class="token punctuation">(</span>addr_sz<span class="token punctuation">)</span><span class="token punctuation">;</span>   <span class="token comment">// CPU Pinned: cudaHostAlloc</span>

<span class="token comment">// 缓冲区大小 (cell.cpp):</span>
UL_ST1<span class="token operator">:</span> UL_ST1_AP_BUF_SIZE x <span class="token function">max</span><span class="token punctuation">(</span>nPuschAnt<span class="token punctuation">,</span> nPucchAnt<span class="token punctuation">,</span> nMaxRxAnt<span class="token punctuation">)</span>
UL_ST2<span class="token operator">:</span> UL_ST2_AP_BUF_SIZE x nSrsAnt
UL_ST3<span class="token operator">:</span> UL_ST3_AP_BUF_SIZE x <span class="token function">max</span><span class="token punctuation">(</span>nPrachAnt<span class="token punctuation">,</span> nMaxRxAnt<span class="token punctuation">)</span>
<span class="token comment">// 每个 Cell 实例化 ul_input_buffer_num_per_cell 个槽位循环使用</span>
</code></pre><p><strong>GDR 零拷贝缓冲区</strong> (<code>gpinned_buffer</code>):</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>alloc_size = rounded_size + GPU_PAGE_SIZE
cuMemAlloc(alloc_size)
  -&gt; cuPointerSetAttribute(SYNC_MEMOPS)
  -&gt; gdr_pin_buffer(rounded_size)   // 内核驱动 pin，消耗 GPU 显存
  -&gt; gdr_map()                      // BAR1 映射到 CPU 用户空间
结果: addr_d (GPU) + addr_h (CPU) 双地址，物理内存只有一份 (GPU 侧)
</code></pre><p><strong>PUSCH 聚合对象同步缓冲区</strong> (<code>phypusch_aggr.cpp</code>):</p>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token function">cudaHostAlloc</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>pFoCompensationBuffersInOut<span class="token punctuation">,</span> size<span class="token punctuation">,</span>
              cudaHostAllocPortable <span class="token operator">|</span> cudaHostAllocMapped<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token comment">// Mapped: GPU 可直接写，CPU 直接读，无需 cudaMemcpy</span>

<span class="token function">cudaHostAlloc</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>pPreEarlyHarqWaitKernelStatus<span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">,</span>
              cudaHostAllocPortable <span class="token operator">|</span> cudaHostAllocMapped<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token comment">// 1 字节标志: GPU kernel 写 0/1 -&gt; CPU 轮询</span>
</code></pre><hr>
<h4 id="922-nvipc--ipc-共享内存池">9.2.2 nvIPC  IPC 共享内存池 </h4>
<p><strong>标准生产配置</strong> (<code>nvipc_5_instances.yaml</code>):</p>
<table>
<thead>
<tr>
<th>池名</th>
<th>buf_size</th>
<th>pool_len</th>
<th>内存类型</th>
<th>合计</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>cpu_msg</code></td>
<td>15,000 B</td>
<td>4,096</td>
<td>CPU (可 pin)</td>
<td>~60 MiB</td>
</tr>
<tr>
<td><code>cpu_data</code></td>
<td>576,000 B</td>
<td>1,024</td>
<td>CPU Pinned</td>
<td>~564 MiB</td>
</tr>
<tr>
<td><code>cpu_large</code></td>
<td>4,096,000 B</td>
<td>64</td>
<td>CPU Pinned</td>
<td>~256 MiB</td>
</tr>
<tr>
<td><code>cuda_data</code></td>
<td>307,200 B</td>
<td>1,024</td>
<td><strong>GPU (cudaMalloc)</strong></td>
<td><strong>~300 MiB</strong></td>
</tr>
<tr>
<td><code>gpu_data</code></td>
<td>可选</td>
<td>可选</td>
<td>GDR (cuMemAlloc)</td>
<td>配置决定</td>
</tr>
</tbody>
</table>
<ul>
<li><code>cpu_data</code> / <code>cpu_large</code> 在 <code>cuda_device_id &gt;= 0</code> 时通过 <code>cudaHostRegister()</code> 转为 pinned memory</li>
<li>DOCA 模式下 <code>cuda_data</code> 使用外部 DOCA buffer (<code>NV_IPC_MEMPOOL_USE_EXT_DOCA_BUFS</code>)</li>
</ul>
<hr>
<h4 id="923-cumac--gpucpu-镜像结构体">9.2.3 cuMAC  GPU/CPU 镜像结构体 </h4>
<p>每个 cell group GPU 端典型字段与大小公式:</p>
<table>
<thead>
<tr>
<th>字段</th>
<th>大小公式</th>
<th>类型</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>estH_fr</code></td>
<td><code>nUe x nBsAnt x nPrbGrp x nUeAnt x 8 B</code></td>
<td>cuComplex 信道矩阵</td>
</tr>
<tr>
<td><code>estH_fr_half</code></td>
<td>同上，<code>2 B</code> 元素</td>
<td>__half 半精度信道矩阵</td>
</tr>
<tr>
<td><code>prdMat</code></td>
<td><code>nCell x nPrbGrp x nBsAnt x maxLayer x 8 B</code></td>
<td>预编码矩阵</td>
</tr>
<tr>
<td><code>sinVal</code> / <code>detMat</code></td>
<td><code>nCell x nPrbGrp x nBsAnt x nUeAnt x 4 B</code></td>
<td>SINR / 行列式</td>
</tr>
<tr>
<td><code>bfGainPrgCurrTx</code></td>
<td><code>nActiveUe x nPrbGrp x 4 B</code></td>
<td>每 PRG 波束赋形增益</td>
</tr>
<tr>
<td><code>allocSol</code></td>
<td><code>2 x nUe x 2 B</code></td>
<td>PRB 分配方案</td>
</tr>
<tr>
<td><code>mcsSelSol</code></td>
<td><code>nUe x 2 B</code></td>
<td>MCS 选择结果</td>
</tr>
<tr>
<td><code>tbErrLast</code></td>
<td><code>nUe x 1 B</code></td>
<td>上次 TB 错误记录</td>
</tr>
<tr>
<td><code>avgRatesActUe</code></td>
<td><code>nActiveUe x 4 B</code></td>
<td>平均速率 float</td>
</tr>
</tbody>
</table>
<p><strong>大规模配置估算</strong> (256 UE, 64 BS ant, 106 PRG, 32 UE ant, 4 cells):</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>estH_fr   256 x 64 x 106 x 32 x 8 B    3.5 GiB GPU
prdMat    4 x 106 x 64 x 4 x 8 B       87  MiB GPU
</code></pre><hr>
<h4 id="924-cuphy-核心--按需-raii-分配">9.2.4 cuPHY 核心  按需 RAII 分配 </h4>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// GPU 设备内存 (每个信道处理 workspace，析构自动释放)</span>
unique_device_ptr<span class="token operator">&lt;</span>T<span class="token operator">&gt;</span>  <span class="token operator">-&gt;</span>  cudaMalloc <span class="token operator">/</span> cudaFree

<span class="token comment">// Host Pinned (TB payload, 测试 bench)</span>
unique_pinned_ptr<span class="token operator">&lt;</span>T<span class="token operator">&gt;</span>  <span class="token operator">-&gt;</span>  cudaHostAlloc <span class="token operator">/</span> cudaFreeHost

<span class="token comment">// GPU Constant Memory (PUCCH, 只写一次，随模块卸载消失)</span>
<span class="token function">cudaMemcpyToSymbolAsync</span><span class="token punctuation">(</span>d_W1<span class="token punctuation">,</span> W1<span class="token punctuation">,</span> <span class="token keyword keyword-sizeof">sizeof</span><span class="token punctuation">(</span>W1<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span>
<span class="token function">cudaMemcpyToSymbolAsync</span><span class="token punctuation">(</span>d_phi_6<span class="token punctuation">,</span> phi_6<span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span>
</code></pre><p>内存用量通过 <code>cuphyMemoryFootprint::addGpuAllocation()</code> 记录。</p>
<hr>
<h4 id="925-aerial-fh-driver--前传-rdma-缓冲区">9.2.5 aerial-fh-driver  前传 RDMA 缓冲区 </h4>
<pre data-role="codeBlock" data-info="" class="language-text"><code>每条 RX/TX 通道分配一个 gpinned_buffer:
  rounded_size = (size_input + GPU_PAGE_SIZE - 1) &amp; GPU_PAGE_MASK
  alloc_size   = rounded_size + GPU_PAGE_SIZE   // 多一页保证对齐

GPU 用量 = sum(alloc_size) x (TX 通道数 + RX 通道数)

回退方案 (无 GDR 硬件):
  cuMemHostAlloc + cuMemHostGetDevicePointer    // Unified Memory，无 BAR1
</code></pre><hr>
<h3 id="93-各模块-cpu-内存用量">9.3 各模块 CPU 内存用量 </h3>
<h4 id="cpu-pinned-memory-dma-able">CPU Pinned Memory (DMA-able) </h4>
<table>
<thead>
<tr>
<th>来源</th>
<th>分配 API</th>
<th>估算大小</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ULInputBuffer::addr_h</code></td>
<td><code>hpinned_alloc</code> (<code>cudaHostAlloc</code>)</td>
<td>与 GPU 侧等量</td>
</tr>
<tr>
<td><code>DLOutputBuffer</code> 主机缓冲</td>
<td><code>hpinned_alloc</code></td>
<td>DL 下行数据大小</td>
</tr>
<tr>
<td><code>nvIPC cpu_data</code></td>
<td><code>mmap</code> + <code>cudaHostRegister</code></td>
<td>~564 MiB</td>
</tr>
<tr>
<td><code>nvIPC cpu_large</code></td>
<td><code>mmap</code> + <code>cudaHostRegister</code></td>
<td>~256 MiB</td>
</tr>
<tr>
<td><code>pFoCompensationBuffersInOut</code></td>
<td><code>cudaHostAllocMapped</code></td>
<td>FO 补偿结果</td>
</tr>
<tr>
<td><code>pPreEarlyHarqWaitKernelStatus</code></td>
<td><code>cudaHostAllocMapped</code></td>
<td>1 字节</td>
</tr>
<tr>
<td><code>cuMAC host mirror</code></td>
<td><code>cudaMallocHost</code></td>
<td>调度结构体 CPU 副本</td>
</tr>
<tr>
<td>CUPTI profiling buffers</td>
<td><code>cudaHostAllocPortable</code></td>
<td>按配置 (默认数 MB)</td>
</tr>
</tbody>
</table>
<h4 id="cpu-regular-memory">CPU Regular Memory </h4>
<table>
<thead>
<tr>
<th>来源</th>
<th>分配</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>schdSolCpu-&gt;allocSol</code></td>
<td><code>new int16_t[]</code></td>
<td>CPU 端调度结果</td>
</tr>
<tr>
<td><code>HarqBucket</code></td>
<td><code>new HarqBucket()</code></td>
<td>16 个桶的元数据</td>
</tr>
<tr>
<td><code>WAvgCfoCache</code></td>
<td>内部 map</td>
<td>CFO 权重缓存结构</td>
</tr>
<tr>
<td>nvIPC ring queues</td>
<td><code>shm_open</code> / <code>mmap</code></td>
<td>消息队列环形缓冲区</td>
</tr>
</tbody>
</table>
<hr>
<h3 id="94-内存用量汇总估算">9.4 内存用量汇总估算 </h3>
<blockquote>
<p>典型 <strong>单 GPU、4 小区、256 UE</strong> 生产配置静态预分配估算。</p>
</blockquote>
<table>
<thead>
<tr>
<th>模块</th>
<th>GPU 内存</th>
<th>CPU Pinned</th>
<th>CPU Regular</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>HARQ 软缓冲池</strong></td>
<td>N x 11.6 MiB</td>
<td></td>
<td></td>
</tr>
<tr>
<td><strong>UL 输入缓冲区</strong></td>
<td>nCell x nSlot x nAnt</td>
<td>与 GPU 等量</td>
<td></td>
</tr>
<tr>
<td><strong>GDR 前传缓冲区</strong></td>
<td>nFlow x FrameSize</td>
<td>(BAR1 共享)</td>
<td></td>
</tr>
<tr>
<td><strong>nvIPC cuda_data</strong></td>
<td>~300 MiB</td>
<td></td>
<td></td>
</tr>
<tr>
<td><strong>nvIPC cpu pools</strong></td>
<td></td>
<td>~880 MiB</td>
<td></td>
</tr>
<tr>
<td><strong>cuMAC 信道矩阵</strong></td>
<td>~3.5 GiB (256 UE)</td>
<td>~调度结果</td>
<td>~调度结构体</td>
</tr>
<tr>
<td><strong>cuPHY workspaces</strong></td>
<td>信道类型 x 配置</td>
<td>~TB payload</td>
<td></td>
</tr>
<tr>
<td><strong>CUDA Context 开销</strong></td>
<td>~50100 MiB x 9 ctx</td>
<td></td>
<td></td>
</tr>
</tbody>
</table>
<p><strong>运行时内存追踪</strong>:</p>
<pre data-role="codeBlock" data-info="bash" class="language-bash bash"><code><span class="token assign-left variable">AERIAL_MEMTRACE</span><span class="token operator">=</span><span class="token number">1</span> ./run_l1.sh

<span class="token comment"># PhyDriverCtx 析构时自动打印:</span>
<span class="token comment">#   Total CPU regular memory  X MiB</span>
<span class="token comment">#   Total CPU pinned memory   X MiB</span>
<span class="token comment">#   Total GPU regular memory  X MiB</span>
<span class="token comment">#   Total GPU pinned memory   X MiB</span>

<span class="token comment"># 实时对账 (代码中插入):</span>
<span class="token comment"># MF_GPU_MEMFOOT_CHECK()  -&gt;  对比 cudaMemGetInfo vs 跟踪计数</span>
</code></pre><hr>
<h3 id="95-内存生命周期">9.5 内存生命周期 </h3>
<pre data-role="codeBlock" data-info="" class="language-text"><code>启动阶段 (一次性预分配):
  PhyDriverCtx 构造 -&gt; MpsCtx x9 -&gt; PhyXxxAggr x N -&gt; HarqPool x3
  Cell 构造 -&gt; ULInputBuffer x (num_per_cell x 3种类型)
  nvIPC open -&gt; cudaMalloc (cuda_data) + cudaHostRegister (cpu pools)

运行阶段 (无动态 GPU 分配):
  HARQ buffer:  从池取出 -&gt; 使用 -&gt; 归还 (lockless ring, 无 cudaMalloc)
  UL buffer:    reserve() -&gt; 处理 -&gt; release() -&gt; cudaMemsetAsync 清零
  nvIPC msg:    tx_allocate -&gt; 填充 -&gt; tx_send -&gt; rx_recv -&gt; rx_release

关闭阶段 (RAII 逆序析构):
  PhyXxxAggr 析构 -&gt; gpinned_buffer: gdr_unmap -&gt; gdr_unpin -&gt; cuMemFree
  HarqPool 析构   -&gt; 所有 HarqBuffer cudaFree
  ULInputBuffer   -&gt; dev_buf + host_buf RAII 自动释放
  nvIPC close     -&gt; cudaFree(cuda_data) + cudaHostUnregister(cpu pools)
  MpsCtx 析构     -&gt; cuCtxDestroy / cuGreenCtxDestroy
</code></pre><hr>
<h3 id="96-注意事项与潜在风险">9.6 注意事项与潜在风险 </h3>
<table>
<thead>
<tr>
<th>风险点</th>
<th>位置</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>cuMAC 逐字段释放易遗漏</td>
<td><code>cumacSubcontext</code> 析构</td>
<td>字段多，新增字段需手动补充释放逻辑</td>
</tr>
<tr>
<td>GDR 释放顺序严格</td>
<td><code>gpinned_buffer</code> 析构</td>
<td>必须 <code>gdr_unmap</code> -&gt; <code>gdr_unpin</code> -&gt; <code>cuMemFree</code>，顺序错误导致内核崩溃</td>
</tr>
<tr>
<td>Mapped memory 对齐</td>
<td>PUSCH aggr</td>
<td>GPU 访问 mapped host memory 须满足 GPU 访问对齐要求</td>
</tr>
<tr>
<td>nvIPC pool 耗尽</td>
<td>运行时</td>
<td><code>cuda_data</code> 1024 槽位，高吞吐场景需监控池耗尽告警</td>
</tr>
<tr>
<td>HARQ pool 耗尽</td>
<td><code>HarqPoolManager</code></td>
<td><code>poolAlloc</code> 返回 nullptr 时软合并失败，退化为 Chase Combining</td>
</tr>
<tr>
<td>MPS 上下文构造失败</td>
<td><code>PhyDriverCtx</code> 构造</td>
<td>中途抛出异常时需确保已创建的 <code>MpsCtx</code> 被正确析构，避免泄漏</td>
</tr>
<tr>
<td>Constant Memory 并发写</td>
<td>PUCCH frontend</td>
<td>多 stream 同时 <code>cudaMemcpyToSymbolAsync</code> 到同一符号存在竞争风险</td>
</tr>
</tbody>
</table>
<hr>
<h2 id="10-内存访问流">10. 内存访问流 </h2>
<p>本节描述 cuPHY 系统中数据从接收天线到 MAC 层的端到端内存访问路径，以及并发拓扑、带宽/延迟参考和优化建议。</p>
<hr>
<h3 id="101-上行-ul-数据路径">10.1 上行 (UL) 数据路径 </h3>
<pre data-role="codeBlock" data-info="" class="language-text"><code>RU 天线原始 IQ 数据
        |  CPRI/eCPRI (Fronthaul 网络)
        v
+------------------------------------------------------------------------+
|  aerial-fh-driver (前传驱动)                                            |
|  gpinned_buffer: addr_d [GPU VRAM/HBM2e] + addr_h [CPU BAR1映射]       |
|  [1] 主路径 GDR/RDMA: NIC DMA --&gt; addr_d [GPU VRAM] (零拷贝)           |
|      回退路径: NIC DMA --&gt; [CPU Pinned] -cudaMemcpy H2D-&gt; [GPU VRAM]   |
+---------------------------+--------------------------------------------+
                            | addr_d [GPU VRAM 指针] 传递给 cuphydriver
                            v
+------------------------------------------------------------------------+
|  cuphydriver -- ULInputBuffer 管理                                      |
|  [2] CPU 端 [CPU Regular Mem]: reserve() 取出空闲槽位 (仅状态位切换)   |
|      GPU 端: 数据已在 addr_d [GPU VRAM]，无需额外拷贝                  |
+---------------------------+--------------------------------------------+
                            | slot addr_d [GPU VRAM 指针] --&gt; puschMpsCtx CUDA stream
                            v
+------------------------------------------------------------------------+
|  PhyPuschAggr -- GPU 信道处理 (puschMpsCtx)                            |
|  [3] FO 补偿: addr_d [GPU VRAM] --&gt; pFoCompensationBuffersInOut         |
|              [CPU Pinned Mapped, cudaHostAllocMapped, GPU可写CPU可读]   |
|      DMRS 信道估计: [GPU VRAM workspace, cudaMalloc RAII]              |
|      均衡+解调: [GPU VRAM workspace] --&gt; LLR buffer [GPU VRAM]         |
|  [4] 早期HARQ: GPU 写 pPreEarlyHarqWaitKernelStatus [CPU Pinned        |
|      Mapped, 1 byte, cudaHostAllocMapped], CPU 轮询读取 [CPU侧]        |
+---------------------------+--------------------------------------------+
                            | LLR [GPU VRAM / GPU Global Memory]
                            v
+------------------------------------------------------------------------+
|  HARQ Pool (GPU Global Memory / GPU VRAM, 启动时 cudaMalloc 预分配)    |
|  [5] 增量冗余: LLR [GPU VRAM] + HarqBuffer[UE_ID][harq_pid] [GPU VRAM]|
|      池选择: Pool[0]=256KiB / Pool[1]=3.8MiB / Pool[2]=7.6MiB         |
|      poolAlloc() 无内存分配, 仅从预分配 GPU 池取指针                    |
+---------------------------+--------------------------------------------+
                            | Decoder 输入 [GPU VRAM / GPU Global Memory]
                            v
+------------------------------------------------------------------------+
|  LDPC Decoder (GPU Kernel, puschMpsCtx stream)                          |
|  [6] 输入/输出均在 [GPU VRAM / GPU Global Memory]                      |
|      解码完成: cudaMemcpyAsync D2H                                      |
|      --&gt; ULInputBuffer::addr_h [CPU Pinned, cudaHostAlloc Portable]    |
+---------------------------+--------------------------------------------+
                            | addr_h [CPU Pinned Memory]
                            v
+------------------------------------------------------------------------+
|  nvIPC -- PHY--&gt;MAC 传递                                                |
|  [7] 路径A: cuda_data 槽 [GPU VRAM, cudaMalloc, 300MiB预分配]          |
|             --&gt; cudaIpcGetMemHandle --&gt; cuMAC 进程 cudaIpcOpenMemHandle |
|      路径B: cpu_data 槽 [CPU Pinned, mmap+cudaHostRegister, 564MiB]    |
|             CPU memcpy addr_h --&gt; SHM --&gt; cuMAC rx_recv [CPU Pinned]   |
+---------------------------+--------------------------------------------+
                            | nvIPC 消息 (SHM ring queue) [CPU Regular/Pinned]
                            v
+------------------------------------------------------------------------+
|  cuMAC -- 上行接收处理                                                  |
|  [8] rx_recv: CPU 从 SHM 取消息 [CPU Regular/Pinned]                   |
|      cudaMemcpy H2D: 数据 --&gt; cumacSubcontext 字段 [GPU VRAM]          |
|      GPU kernel: 速率计算、HARQ ACK/NACK 生成 [GPU VRAM]              |
|      cudaMemcpy D2H: 结果 [GPU VRAM] --&gt; CPU mirror struct [CPU Pinned]|
+------------------------------------------------------------------------+
</code></pre><p><strong>关键路径内存访问跳数 (UL)</strong>:</p>
<table>
<thead>
<tr>
<th>跳数</th>
<th>源</th>
<th>源内存类型</th>
<th>目标</th>
<th>目标内存类型</th>
<th>方式</th>
<th>触发者</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>NIC (RU 数据)</td>
<td>—</td>
<td>gpinned_buffer.addr_d</td>
<td><strong>GPU VRAM</strong> (HBM2e)</td>
<td>RDMA/DMA 零拷贝</td>
<td>NIC DMA 引擎</td>
</tr>
<tr>
<td>1'</td>
<td>NIC (回退)</td>
<td>—</td>
<td>ULInputBuffer::addr_h</td>
<td><strong>CPU Pinned</strong></td>
<td>NIC DMA</td>
<td>NIC DMA 引擎</td>
</tr>
<tr>
<td>1''</td>
<td>回退接续</td>
<td><strong>CPU Pinned</strong></td>
<td>ULInputBuffer::addr_d</td>
<td><strong>GPU VRAM</strong></td>
<td>cudaMemcpy H2D</td>
<td>CUDA DMA</td>
</tr>
<tr>
<td>2</td>
<td>addr_d</td>
<td><strong>GPU VRAM</strong></td>
<td>GPU Workspace</td>
<td><strong>GPU VRAM</strong></td>
<td>GPU Kernel 读</td>
<td>CUDA kernel</td>
</tr>
<tr>
<td>3</td>
<td>GPU Workspace (LLR)</td>
<td><strong>GPU VRAM</strong></td>
<td>HarqBuffer[UE][pid]</td>
<td><strong>GPU VRAM</strong></td>
<td>GPU Kernel 写</td>
<td>CUDA kernel</td>
</tr>
<tr>
<td>4</td>
<td>HARQ Pool</td>
<td><strong>GPU VRAM</strong></td>
<td>LDPC Decoder 输入</td>
<td><strong>GPU VRAM</strong></td>
<td>GPU 内部传输</td>
<td>CUDA kernel</td>
</tr>
<tr>
<td>5</td>
<td>LDPC 解码结果</td>
<td><strong>GPU VRAM</strong></td>
<td>ULInputBuffer::addr_h</td>
<td><strong>CPU Pinned</strong></td>
<td>cudaMemcpyAsync D2H</td>
<td>CUDA DMA 引擎</td>
</tr>
<tr>
<td>6</td>
<td>addr_h</td>
<td><strong>CPU Pinned</strong></td>
<td>nvIPC cpu_data 槽</td>
<td><strong>CPU Pinned</strong></td>
<td>CPU memcpy</td>
<td>CPU 线程</td>
</tr>
<tr>
<td>6'</td>
<td>addr_d (路径A)</td>
<td><strong>GPU VRAM</strong></td>
<td>nvIPC cuda_data 槽</td>
<td><strong>GPU VRAM</strong></td>
<td>CUDA IPC 句柄传递</td>
<td>CPU 线程</td>
</tr>
<tr>
<td>7</td>
<td>nvIPC SHM/CUDA IPC</td>
<td><strong>CPU Pinned / GPU VRAM</strong></td>
<td>cuMAC GPU 字段</td>
<td><strong>GPU VRAM</strong></td>
<td>cudaMemcpy H2D</td>
<td>cuMAC CPU 线程</td>
</tr>
</tbody>
</table>
<hr>
<h3 id="102-下行-dl-数据路径">10.2 下行 (DL) 数据路径 </h3>
<pre data-role="codeBlock" data-info="" class="language-text"><code>cuMAC 调度器 (CPU)
        |  调度结果: allocSol, mcsSelSol (GPU mirror struct)
        v
+------------------------------------------------------------------------+
|  cuMAC --&gt; nvIPC --&gt; cuphydriver                                        |
|  [1] 路径A: cuMAC GPU mirror [GPU VRAM] -- cudaMemcpy D2H --&gt;          |
|             CPU Pinned (Host mirror) [CPU Pinned, cudaMallocHost]      |
|             --&gt; nvIPC cpu_data 槽 [CPU Pinned] (SHM tx_send)           |
|      路径B: cuMAC GPU mirror [GPU VRAM] --&gt; nvIPC cuda_data 槽         |
|             [GPU VRAM] via cudaIpcGetMemHandle (无 D2H 拷贝)           |
+---------------------------+--------------------------------------------+
                            | nvIPC 消息 [CPU Pinned SHM / GPU VRAM IPC]
                            v
+------------------------------------------------------------------------+
|  cuphydriver -- DL 调度接收                                             |
|  [2] rx_recv: 从 nvIPC 取调度结果 + TB payload                         |
|      DL Buffer reserve(): 取空闲 DL 槽位 [CPU Regular状态位]           |
|      cudaMemcpyAsync H2D: TB payload [CPU Pinned]                      |
|                           --&gt; DLBuffer.addr_d [GPU VRAM]               |
+---------------------------+--------------------------------------------+
                            | DLBuffer.addr_d [GPU VRAM 指针]
                            v
+------------------------------------------------------------------------+
|  PhyPdschAggr -- GPU 下行处理 (pdschMpsCtx, priority -4)              |
|  [3] PDSCH 编码: DLBuffer [GPU VRAM] --&gt; LDPC 编码 [GPU VRAM]         |
|      速率匹配: [GPU VRAM] --&gt; 比特缓冲 [GPU VRAM]                      |
|      调制映射: [GPU VRAM] --&gt; 复数符号 [GPU VRAM]                      |
|      预编码: prdMat [GPU VRAM, cuMAC分配] x 信号 --&gt; 天线IQ [GPU VRAM]|
+---------------------------+--------------------------------------------+
                            | 天线端口 IQ 数据 [GPU VRAM / GPU Global Memory]
                            v
+------------------------------------------------------------------------+
|  aerial-fh-driver -- 下行发送                                           |
|  [4] gpinned_buffer: addr_d [GPU VRAM] + addr_h [CPU BAR1映射]         |
|      GPU kernel 写天线 IQ --&gt; addr_d [GPU VRAM]                        |
|      主路径 GDR/RDMA: NIC DMA 从 addr_d [GPU VRAM] 直读 --&gt; eCPRI发送  |
|      回退: [GPU VRAM] -cudaMemcpy D2H-&gt; [CPU Pinned] --&gt; NIC DMA      |
+------------------------------------------------------------------------+
                            | eCPRI / CPRI --&gt; RU 天线
</code></pre><p><strong>关键路径内存访问跳数 (DL)</strong>:</p>
<table>
<thead>
<tr>
<th>跳数</th>
<th>源</th>
<th>源内存类型</th>
<th>目标</th>
<th>目标内存类型</th>
<th>方式</th>
<th>触发者</th>
</tr>
</thead>
<tbody>
<tr>
<td>1A</td>
<td>cuMAC GPU mirror</td>
<td><strong>GPU VRAM</strong></td>
<td>CPU Pinned Host mirror</td>
<td><strong>CPU Pinned</strong></td>
<td>cudaMemcpy D2H</td>
<td>CPU 线程</td>
</tr>
<tr>
<td>1B</td>
<td>CPU Pinned Host mirror</td>
<td><strong>CPU Pinned</strong></td>
<td>nvIPC cpu_data 槽</td>
<td><strong>CPU Pinned</strong></td>
<td>CPU memcpy / SHM</td>
<td>CPU 线程</td>
</tr>
<tr>
<td>1'</td>
<td>cuMAC GPU mirror</td>
<td><strong>GPU VRAM</strong></td>
<td>nvIPC cuda_data 槽</td>
<td><strong>GPU VRAM</strong></td>
<td>CUDA IPC 句柄</td>
<td>CPU 线程 (路径B，无D2H)</td>
</tr>
<tr>
<td>2</td>
<td>nvIPC 槽 (TB payload)</td>
<td><strong>CPU Pinned</strong></td>
<td>DLBuffer.addr_d</td>
<td><strong>GPU VRAM</strong></td>
<td>cudaMemcpyAsync H2D</td>
<td>CUDA DMA 引擎</td>
</tr>
<tr>
<td>3</td>
<td>DLBuffer.addr_d</td>
<td><strong>GPU VRAM</strong></td>
<td>LDPC/调制/预编码 workspace</td>
<td><strong>GPU VRAM</strong></td>
<td>GPU Kernel 读</td>
<td>CUDA kernel</td>
</tr>
<tr>
<td>4</td>
<td>Workspace / prdMat</td>
<td><strong>GPU VRAM</strong></td>
<td>天线 IQ buffer (addr_d)</td>
<td><strong>GPU VRAM</strong></td>
<td>GPU Kernel 写</td>
<td>CUDA kernel</td>
</tr>
<tr>
<td>5</td>
<td>addr_d</td>
<td><strong>GPU VRAM</strong> (HBM2e)</td>
<td>NIC</td>
<td>—</td>
<td>GDR RDMA 零拷贝</td>
<td>NIC DMA 引擎</td>
</tr>
<tr>
<td>5'</td>
<td>addr_d (回退)</td>
<td><strong>GPU VRAM</strong></td>
<td>CPU Pinned 回退缓冲</td>
<td><strong>CPU Pinned</strong></td>
<td>cudaMemcpy D2H</td>
<td>CUDA DMA 引擎</td>
</tr>
</tbody>
</table>
<hr>
<h3 id="103-srs-跨进程内存流">10.3 SRS 跨进程内存流 </h3>
<p>SRS (Sounding Reference Signal) 处理跨越 UL 接收和波束赋形权重更新两个流程:</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>RU IQ 数据 (eCPRI 帧)
    | GDR/RDMA --&gt; gpinned_buffer.addr_d
    v
gpinned_buffer.addr_d [GPU VRAM, cuMemAlloc + gdr_pin_buffer]
    | srsMpsCtx stream (priority -2)
    v
PhySrsAggr -- GPU SRS 处理
    | 输入: addr_d [GPU VRAM]
    | SRS 信道估计 workspace [GPU VRAM, cudaMalloc RAII]
    | 结果写入: BFW buffer [GPU VRAM, Cell配置时cudaMalloc预分配]
    v
PhyUlBfwAggr (ulBfwMpsCtx, priority -2)
    | [1] BFW 权重更新:
    |     读: BFW buffer [GPU VRAM]
    |     写: cuMAC prdMat [GPU VRAM] / bfGainPrgCurrTx [GPU VRAM]
    |     跨 MPS context 同步 (两个 GPU VRAM context 间):
    |     cudaEventRecord(srsDoneEvent) -&gt; cudaStreamWaitEvent(ulBfwStream)
    v
cuMAC prdMat [GPU VRAM, cumacSubcontext cudaMalloc]
    | [2] cudaMemcpy D2H:
    |     prdMat [GPU VRAM] --&gt; CPU mirror [CPU Pinned, cudaMallocHost]
    |     --&gt; 下次 DL 调度读取预编码矩阵 (CPU 侧)
    v
PhyPdschAggr 预编码:
    读取 prdMat [GPU VRAM] x 信号 [GPU VRAM] --&gt; 天线IQ [GPU VRAM]
</code></pre><p><strong>SRS 内存对象生命周期</strong>:</p>
<table>
<thead>
<tr>
<th>对象</th>
<th>分配 API</th>
<th>分配时机</th>
<th>释放时机</th>
<th>驻留位置</th>
</tr>
</thead>
<tbody>
<tr>
<td>SRS IQ input (addr_d)</td>
<td><code>cuMemAlloc</code> + <code>gdr_pin_buffer</code></td>
<td>Cell 配置时预分配</td>
<td>下一 slot 循环复用</td>
<td><strong>GPU VRAM</strong> (gpinned_buffer, BAR1同时映射至CPU)</td>
</tr>
<tr>
<td>SRS IQ input (addr_h)</td>
<td><code>gdr_map</code> BAR1映射</td>
<td>同上</td>
<td>同上</td>
<td><strong>CPU BAR1映射</strong> (不占用CPU DRAM)</td>
</tr>
<tr>
<td>SRS 信道估计 workspace</td>
<td><code>cudaMalloc</code> (RAII)</td>
<td>PhySrsAggr 构造</td>
<td>PhySrsAggr 析构</td>
<td><strong>GPU VRAM</strong></td>
</tr>
<tr>
<td>BFW 权重结果</td>
<td><code>cudaMalloc</code></td>
<td>Cell 配置时预分配</td>
<td>Cell 销毁</td>
<td><strong>GPU VRAM</strong></td>
</tr>
<tr>
<td>cuMAC prdMat (GPU侧)</td>
<td><code>cudaMalloc</code> (逐字段)</td>
<td>cumacSubcontext 构造</td>
<td>cumacSubcontext 析构</td>
<td><strong>GPU VRAM</strong></td>
</tr>
<tr>
<td>cuMAC prdMat (CPU镜像)</td>
<td><code>cudaMallocHost</code></td>
<td>cumacSubcontext 构造</td>
<td>cumacSubcontext 析构</td>
<td><strong>CPU Pinned</strong></td>
</tr>
</tbody>
</table>
<hr>
<h3 id="104-带宽与延迟参考">10.4 带宽与延迟参考 </h3>
<blockquote>
<p>以 NVIDIA A100 80GB SXM / ConnectX-7 RoCEv2 / x86 Xeon 为典型硬件基准。</p>
</blockquote>
<h4 id="1041-内存带宽参考">10.4.1 内存带宽参考 </h4>
<table>
<thead>
<tr>
<th>路径</th>
<th>峰值带宽</th>
<th>典型实测</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>GPU HBM 读/写</td>
<td>2,039 GB/s</td>
<td>~1,800 GB/s</td>
<td>A100 HBM2e</td>
</tr>
<tr>
<td>PCIe 5.0 x16 H2D/D2H</td>
<td>128 GB/s</td>
<td>60–80 GB/s</td>
<td>双向，实测受协议开销影响</td>
</tr>
<tr>
<td>GPUDirect RDMA (BAR1)</td>
<td>50–100 GB/s</td>
<td>25–50 GB/s</td>
<td>受 BAR1 窗口大小限制 (通常 16–64 GiB)</td>
</tr>
<tr>
<td>NVLink 3.0 (多 GPU)</td>
<td>600 GB/s</td>
<td>~500 GB/s</td>
<td>A100 NVLink (本项目未使用)</td>
</tr>
<tr>
<td>DDR5 Host Memory</td>
<td>307 GB/s</td>
<td>~250 GB/s</td>
<td>双通道 DDR5-4800</td>
</tr>
<tr>
<td>100GbE eCPRI 前传</td>
<td>12.5 GB/s</td>
<td>10–11 GB/s</td>
<td>含 eCPRI 协议头开销</td>
</tr>
</tbody>
</table>
<h4 id="1042-延迟参考">10.4.2 延迟参考 </h4>
<table>
<thead>
<tr>
<th>操作</th>
<th>典型延迟</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>GPU kernel 启动</td>
<td>5–20 us</td>
<td>含 CUDA driver 调度开销</td>
</tr>
<tr>
<td>cudaMemcpy H2D (4 KiB)</td>
<td>10–30 us</td>
<td>含 PCIe 往返 + driver overhead</td>
</tr>
<tr>
<td>cudaMemcpy H2D (4 MiB)</td>
<td>50–200 us</td>
<td>PCIe 传输主导</td>
</tr>
<tr>
<td>GDR Write CPU-&gt;GPU (4 KiB)</td>
<td>2–5 us</td>
<td>BAR1 映射写，无需 kernel</td>
</tr>
<tr>
<td>GDR Read CPU&lt;-GPU (4 KiB)</td>
<td>3–8 us</td>
<td>BAR1 映射读</td>
</tr>
<tr>
<td>RDMA Write NIC-&gt;GPU (4 KiB)</td>
<td>1–3 us</td>
<td>GPUDirect RDMA，含 IB 往返</td>
</tr>
<tr>
<td>cudaEventRecord + Wait</td>
<td>1–5 us</td>
<td>stream 间同步开销</td>
</tr>
<tr>
<td>nvIPC SHM tx-&gt;rx</td>
<td>1–10 us</td>
<td>同机 POSIX SHM + 信号量</td>
</tr>
<tr>
<td>5G NR slot 时长 (30 kHz SCS)</td>
<td>500 us</td>
<td>L1 处理预算: &lt; 250 us</td>
</tr>
</tbody>
</table>
<h4 id="1043-cuphy-l1-处理时间预算">10.4.3 cuPHY L1 处理时间预算 </h4>
<pre data-role="codeBlock" data-info="" class="language-text"><code>slot N 到达:             t = 0
  FH 接收完成:           t ~ 0-50 us    (eCPRI 帧聚合)
  UL 信道处理启动:       t ~ 50 us
  PUSCH 解调完成:        t ~ 50-150 us  (GPU 处理)
  HARQ 软合并完成:       t ~ 150-180 us
  LDPC 解码完成:         t ~ 180-220 us
  D2H 结果传回:          t ~ 220-240 us
  nvIPC --&gt; cuMAC 接收:  t ~ 240-250 us
  MAC 调度 (slot N+k):   t ~ 250 us+    (DL 处理预算)
  DL 发送截止:           t = 500 us     (1 slot = 500 us @ 30 kHz SCS)
</code></pre><hr>
<h3 id="105-并发内存访问拓扑">10.5 并发内存访问拓扑 </h3>
<p>下图展示多个 MPS 上下文并发执行时的内存访问关系:</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>                 ┌──────────────────────────────────────────────┐
                 │         GPU VRAM (HBM2e) — 共享物理显存       │
                 │  ULBuf[GPU]  HARQPool[GPU]  prdMat[GPU]      │
                 │  DLBuf[GPU]  BFW[GPU]       nvIPC cuda[GPU]  │
                 └───────────────┬──────────────────────────────┘
                                 │ 各 MPS Context 各自拥有独立 CUcontext
          ┌──────────────────────┼────────────────────┐
          │                      │                    │
          v                      v                    v
+-----------+          +--------------+       +--------------+
|pdschMpsCtx| [GPU]    |puschMpsCtx   | [GPU] |srsMpsCtx     | [GPU]
|priority-4 |          |priority-2    |       |priority-2    |
|           |          |              |       |              |
|R:DLBuf    | [GPU]    |R:ULBuf       | [GPU] |R:ULBuf       | [GPU]
|W:AntIQ    | [GPU]    |W:LLR         | [GPU] |W:BFW         | [GPU]
|R:prdMat   | [GPU]    |RW:HARQPool   | [GPU] |              |
+-----------+          +--------------+       +--------------+

+-----------+          +--------------+       +--------------+
|pucchMpsCtx| [GPU]    |prachMpsCtx   | [GPU] |ulBfwMpsCtx   | [GPU]
|priority-3 |          |priority-3    |       |priority-2    |
|           |          |              |       |              |
|R:ULBuf    | [GPU]    |R:ULBuf       | [GPU] |R:SRS BFW     | [GPU]
|W:UCI      | [GPU]    |W:PRACH Res   | [GPU] |W:prdMat      | [GPU]
+-----------+          +--------------+       +--------------+

CPU 侧同步对象 (不在 GPU VRAM 中):
  reserve()/release() 状态位  [CPU Regular Memory]
  pPreEarlyHarqWaitKernelStatus [CPU Pinned Mapped, cudaHostAllocMapped]
  pFoCompensationBuffersInOut   [CPU Pinned Mapped, cudaHostAllocMapped]
  nvIPC SHM ring queue          [CPU Pinned, mmap+cudaHostRegister]
</code></pre><p><strong>共享内存对象并发访问说明</strong>:</p>
<table>
<thead>
<tr>
<th>对象</th>
<th>内存类型</th>
<th>并发访问者</th>
<th>竞争情况</th>
<th>保护机制</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ULInputBuffer::addr_d</code></td>
<td><strong>GPU VRAM</strong></td>
<td>PUSCH/PUCCH/PRACH/SRS 多 context 并发读</td>
<td>GPU kernel 只读，无写竞争</td>
<td>无需 GPU 锁</td>
</tr>
<tr>
<td><code>ULInputBuffer</code> slot 状态</td>
<td><strong>CPU Regular</strong></td>
<td>CPU 多线程 reserve()/release()</td>
<td>有竞争</td>
<td>CPU mutex/原子操作</td>
</tr>
<tr>
<td><code>HARQ Pool</code> (HarqBuffer)</td>
<td><strong>GPU VRAM</strong></td>
<td>puschMpsCtx 独占</td>
<td>per-UE 独立 buffer，无竞争</td>
<td>Lockless ring (per-pool)</td>
</tr>
<tr>
<td><code>nvIPC cuda_data</code> 槽</td>
<td><strong>GPU VRAM</strong></td>
<td>多 producer/consumer</td>
<td>有竞争</td>
<td>Lockless ring buffer</td>
</tr>
<tr>
<td><code>nvIPC cpu_data</code> 槽</td>
<td><strong>CPU Pinned</strong></td>
<td>多 producer/consumer</td>
<td>有竞争</td>
<td>POSIX 信号量</td>
</tr>
<tr>
<td><code>cuMAC prdMat</code> (GPU)</td>
<td><strong>GPU VRAM</strong></td>
<td>ulBfwMpsCtx 写 / pdschMpsCtx 读</td>
<td>slot 边界有竞争</td>
<td>slot 边界 cudaEvent 同步</td>
</tr>
<tr>
<td><code>cuMAC prdMat</code> (CPU mirror)</td>
<td><strong>CPU Pinned</strong></td>
<td>CPU 调度线程读</td>
<td>无并发写</td>
<td>D2H 完成后 CPU 独占读</td>
</tr>
<tr>
<td><code>pPreEarlyHarqWaitKernelStatus</code></td>
<td><strong>CPU Pinned Mapped</strong></td>
<td>GPU 写 / CPU 轮询读</td>
<td>GPU写CPU读，无写-写竞争</td>
<td>硬件 cache coherency</td>
</tr>
<tr>
<td><code>pFoCompensationBuffersInOut</code></td>
<td><strong>CPU Pinned Mapped</strong></td>
<td>GPU 写 / CPU 读</td>
<td>GPU写CPU读</td>
<td>硬件 cache coherency</td>
</tr>
</tbody>
</table>
<p><strong>同步机制汇总</strong>:</p>
<table>
<thead>
<tr>
<th>场景</th>
<th>涉及内存类型</th>
<th>机制</th>
<th>对象</th>
</tr>
</thead>
<tbody>
<tr>
<td>跨 stream GPU 数据依赖</td>
<td>GPU VRAM → GPU VRAM</td>
<td><code>cudaEventRecord</code> + <code>cudaStreamWaitEvent</code></td>
<td>srsDoneEvent → ulBfwStream</td>
</tr>
<tr>
<td>CPU 等待 GPU 写结果</td>
<td>GPU VRAM → <strong>CPU Pinned Mapped</strong></td>
<td>1字节 Mapped Pinned 轮询</td>
<td><code>pPreEarlyHarqWaitKernelStatus</code></td>
</tr>
<tr>
<td>UL Buffer slot 管理</td>
<td><strong>CPU Regular</strong> 状态位</td>
<td>CPU 端原子操作 / mutex</td>
<td><code>ULInputBuffer::reserve()</code></td>
</tr>
<tr>
<td>HARQ Pool 无锁分配</td>
<td><strong>GPU VRAM</strong> 池指针</td>
<td>Lockless ring (per-pool)</td>
<td><code>HarqPoolManager::poolAlloc()</code></td>
</tr>
<tr>
<td>PHY→MAC 消息传递</td>
<td><strong>CPU Pinned</strong> SHM / <strong>GPU VRAM</strong> IPC</td>
<td>POSIX 信号量 / CUDA IPC</td>
<td>nvIPC ring queue head/tail</td>
</tr>
<tr>
<td>MPS SM 资源隔离</td>
<td>GPU SM 硬件资源</td>
<td>CUDA Driver 硬件调度</td>
<td>SM Partition per MpsCtx</td>
</tr>
</tbody>
</table>
<hr>
<h3 id="106-热点分析与优化建议">10.6 热点分析与优化建议 </h3>
<table>
<thead>
<tr>
<th>#</th>
<th>热点位置</th>
<th>涉及内存类型</th>
<th>问题描述</th>
<th>优化建议</th>
<th>预期收益</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td><code>ULInputBuffer</code> H2D</td>
<td><strong>CPU Pinned</strong> → <strong>GPU VRAM</strong> (PCIe)</td>
<td>无 GDR 时每 slot PUSCH 数据 H2D 拷贝占用 PCIe 带宽</td>
<td>启用 GDR 零拷贝（NIC DMA 直达 GPU VRAM）；无 GDR 时优先 RDMA 网卡</td>
<td>PCIe 带宽节省 60–80%</td>
</tr>
<tr>
<td>2</td>
<td><code>cuMAC cumacSubcontext</code> 初始化</td>
<td><strong>GPU VRAM</strong> (cudaMalloc)</td>
<td>逐字段 <code>cudaMalloc</code> 初始化慢（数百次 CUDA API 调用，每次有 driver 开销）</td>
<td>单次大块 <code>cudaMalloc</code> + 偏移指针切分字段，减少 API 调用轮次</td>
<td>启动时间缩短 30–50%</td>
</tr>
<tr>
<td>3</td>
<td>HARQ Pool 预分配</td>
<td><strong>GPU VRAM</strong></td>
<td><code>Pool[2]</code> (7.6 MiB/buffer × N) 大量预占 GPU VRAM，低负载时浪费</td>
<td>引入动态扩缩容；或按实际 cell/UE 配置裁剪 Pool 数量</td>
<td>GPU VRAM 节省 20–40%</td>
</tr>
<tr>
<td>4</td>
<td>nvIPC <code>cpu_data</code> 池</td>
<td><strong>CPU Pinned</strong> (~564 MiB 常驻)</td>
<td>大量 pinned memory 常驻，TLB entry 压力大，影响 CPU 侧 cache 效率</td>
<td>评估改用 <code>cuda_data</code> (<strong>GPU VRAM</strong>) 全程 GPU IPC 路径，跳过 CPU pinned</td>
<td>CPU-GPU 同步延迟降低</td>
</tr>
<tr>
<td>5</td>
<td>cuMAC GPU→CPU mirror D2H</td>
<td><strong>GPU VRAM</strong> → <strong>CPU Pinned</strong></td>
<td>每次调度 D2H 传输全量镜像结构体（含未变更字段）</td>
<td>仅传输 dirty 字段（增量同步）；或改用 <code>cudaHostAllocMapped</code> <strong>CPU Pinned Mapped</strong> 直接共享，消除拷贝</td>
<td>D2H 带宽降低 50–70%</td>
</tr>
<tr>
<td>6</td>
<td>PUCCH Constant Memory 并发写</td>
<td><strong>GPU Constant Memory</strong></td>
<td>多 stream <code>cudaMemcpyToSymbolAsync</code> 同时写同一 <code>__constant__</code> 符号存在竞争</td>
<td>改为模块加载时一次性写入；或改用 <code>__device__</code> <strong>GPU VRAM</strong> + L2 cache pin</td>
<td>消除运行时写竞争</td>
</tr>
<tr>
<td>7</td>
<td>MPS Context × 9</td>
<td><strong>GPU VRAM</strong> (~50–100 MiB/ctx)</td>
<td>每个 CUcontext 本身占用约 50–100 MiB GPU VRAM（page table、堆栈等）</td>
<td>合并低负载 context（如 <code>dlBfw</code> + <code>pdcch</code>）；使用 Green Context 共享 SM 且降低 context 数</td>
<td>GPU VRAM 节省 200–400 MiB</td>
</tr>
<tr>
<td>8</td>
<td>GDR BAR1 窗口超限</td>
<td><strong>GPU VRAM</strong> ↔ BAR1 映射</td>
<td>A100 BAR1 默认 64 GiB；多 <code>gpinned_buffer</code> 累计超限时 <code>gdr_pin_buffer</code> 静默回退至 <strong>CPU Pinned</strong>，引入额外 PCIe 拷贝</td>
<td>提前统计所有 <code>gpinned_buffer</code> 总量；监控 <code>gdr_pin_buffer</code> 返回码；超限时显式告警并拒绝新分配</td>
<td>避免延迟抖动和隐式 PCIe 拷贝</td>
</tr>
<tr>
<td>9</td>
<td>LDPC Decoder workspace</td>
<td><strong>GPU VRAM</strong> (动态 cudaMalloc)</td>
<td>若 workspace 未池化，每 slot <code>cudaMalloc</code>/<code>cudaFree</code> 引入 CUDA driver 锁开销（~5–20 µs）</td>
<td>在 <code>PhyPuschAggr</code> 构造时预分配最大 workspace (<strong>GPU VRAM</strong>)，运行时复用，无需 <code>cudaMalloc</code></td>
<td>消除每 slot <code>cudaMalloc</code> 延迟</td>
</tr>
<tr>
<td>10</td>
<td><code>pFoCompensationBuffersInOut</code></td>
<td><strong>CPU Pinned Mapped</strong> (GPU写/CPU读)</td>
<td>GPU 高频写、CPU 频繁读，导致 CPU LLC 被污染（Mapped Pinned 绕过 CPU cache coherency 协议）</td>
<td>CPU 读时使用 non-temporal load（<code>_mm_stream_load_si128</code>）避免 LLC 污染；或改用 <strong>GPU VRAM</strong> + 异步 D2H</td>
<td>LLC 缓存命中率提升</td>
</tr>
</tbody>
</table>
<hr>
<h2 id="11-8cc-场景内存访问超时分析">11. 8CC 场景内存访问超时分析 </h2>
<p>本章基于第 10 章梳理的内存访问流，结合 <code>cuphycontroller_F08.yaml</code>（8 小区组网参考配置）与驱动源码，分析 <strong>8CC（8 Component Carrier，8 载波/小区）</strong> 场景下出现"内存访问超时"的可能原因。所有结论均以实际代码与配置为依据。</p>
<blockquote>
<p>说明：本章所说的"内存访问超时"，在 cuPHY-CP 中具体表现为以下两类可观测事件，它们都直接或间接由 <strong>GPU/CPU 内存访问被阻塞</strong> 引起：</p>
<ol>
<li><strong>Order Kernel 超时</strong>：<code>ORDER_KERNEL_EXIT_TIMEOUT_RX_PKT</code> / <code>ORDER_KERNEL_EXIT_TIMEOUT_NO_PKT</code>（GPU 持久核轮询接收内存超时）。</li>
<li><strong>L2/L2A 晚 slot 丢弃</strong>：<code>check_time_threshold()</code> 判定 <code>&gt; 500µs</code> → Drop slot → FAPI <code>SCF_ERROR_CODE_MSG_LATE_SLOT_ERR (0x34)</code>。</li>
</ol>
</blockquote>
<h3 id="111-超时的定义与触发链路源码锚点">11.1 超时的定义与触发链路（源码锚点） </h3>
<p><strong>(1) Order Kernel 超时（GPU 侧轮询接收内存）</strong></p>
<p>Order Kernel 是常驻 GPU 的"持久核"（persistent kernel），自旋轮询 NIC 写入的接收缓冲区（GDR 路径为 <strong>GPU VRAM</strong>，CPU 路径为 <strong>CPU Pinned</strong>），等待 PUSCH/PRACH/SRS 的 U-plane 报文到齐后再触发后续 cuPHY 流水线。其超时退出条件在 <code>order_cuda_kernels.cu</code> 中通过 volatile 写或 <code>atomicCAS</code> 置位：</p>
<pre data-role="codeBlock" data-info="cuda" class="language-cuda cuda"><code>// order_cuda_kernels.cu （多处）
DOCA_GPUNETIO_VOLATILE(*exit_cond_d_cell) = ORDER_KERNEL_EXIT_TIMEOUT_RX_PKT;   // 已收到部分包但未集齐，超时
DOCA_GPUNETIO_VOLATILE(*exit_cond_d_cell) = ORDER_KERNEL_EXIT_TIMEOUT_NO_PKT;   // 完全未收到包，超时
// order_cuda_kernels.cu:3932 / 3971（aggr 模式用原子操作）
atomicCAS(exit_cond_d_cell, ORDER_KERNEL_RUNNING, ORDER_KERNEL_EXIT_TIMEOUT_RX_PKT);
</code></pre><p>超时阈值来自 <code>context.cpp:271-291</code>（单位 ns）：</p>
<table>
<thead>
<tr>
<th>超时参数</th>
<th>计算/默认</th>
<th>F08(8CC) 取值</th>
<th>含义</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ul_order_timeout_gpu_ns</code></td>
<td>配置值，缺省 <code>ORDER_KERNEL_WAIT_TIMEOUT_MS×NS_X_MS</code>(8ms)</td>
<td><strong>2,300,000 ns (2.3ms)</strong></td>
<td>整 slot 收包等待上限（GPU 持久核轮询 <strong>接收内存</strong>）</td>
</tr>
<tr>
<td><code>ul_order_timeout_first_pkt_gpu_ns</code></td>
<td><code>= ul_order_timeout_gpu_ns / 2</code></td>
<td><strong>1,150,000 ns (1.15ms)</strong></td>
<td>首包等待上限（无任何包到达即判 <code>NO_PKT</code>）</td>
</tr>
<tr>
<td><code>ul_order_timeout_gpu_srs_ns</code></td>
<td>同上(SRS 专用)</td>
<td>配置值</td>
<td>SRS Order Kernel 等待上限</td>
</tr>
<tr>
<td><code>ul_order_timeout_cpu_ns</code></td>
<td>缺省 <code>ORDER_KERNEL_ENABLE_THRESHOLD×NS_X_MS</code></td>
<td><strong>8,000,000 ns (8ms)</strong></td>
<td>CPU 侧 Order 等待上限</td>
</tr>
<tr>
<td><code>ul_order_max_rx_pkts</code></td>
<td>缺省 <code>ORDER_KERNEL_MAX_RX_PKTS</code></td>
<td>—</td>
<td>单次轮询最多收包数</td>
</tr>
<tr>
<td><code>ul_order_rx_pkts_timeout_ns</code></td>
<td>缺省 <code>ORDER_KERNEL_RX_PKTS_TIMEOUT_NS</code></td>
<td>—</td>
<td>相邻两包之间的等待上限</td>
</tr>
</tbody>
</table>
<p>超时后由 CPU 侧 <code>task_function_ul_aggr.cpp:2404</code> 捕获并尝试终止 PUSCH 流水线：</p>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// task_function_ul_aggr.cpp:2404</span>
<span class="token function">NVLOGE_FMT</span><span class="token punctuation">(</span>TAG<span class="token punctuation">,</span> AERIAL_CUPHY_API_EVENT<span class="token punctuation">,</span> <span class="token string">"... Order kernel timeout error (exit condition {}) for cell index {} ..."</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token comment">// 连续超时累加，超过阈值即把该 cell 标记为 unhealthy（2413-2419）</span>
cell_ptr<span class="token operator">-&gt;</span>num_consecutive_ok_timeout<span class="token operator">++</span><span class="token punctuation">;</span>
<span class="token keyword keyword-if">if</span><span class="token punctuation">(</span>ok_timeout <span class="token operator">&gt;=</span> pdctx<span class="token operator">-&gt;</span><span class="token function">getAggr_obj_non_avail_th</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">/</span><span class="token number">2</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> cell_ptr<span class="token operator">-&gt;</span><span class="token function">setUnhealthy</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token punctuation">}</span>
</code></pre><p><strong>(2) L2/L2A 晚 slot 丢弃（CPU 侧截止期）</strong></p>
<p>即使 GPU 未触发 Order 超时，只要整条流水线（含上文所有内存搬运）使 L2+L2A 处理超过 slot 截止期，<code>nv_phy_module.cpp:1007</code> 即丢弃该 slot：</p>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// nv_phy_module.cpp:1007  check_time_threshold()</span>
<span class="token keyword keyword-if">if</span><span class="token punctuation">(</span><span class="token punctuation">(</span>now <span class="token operator">-</span> l1_slot_ind_tick_<span class="token punctuation">[</span>slot<span class="token operator">%</span><span class="token number">10</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token function">count</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&gt;</span> <span class="token punctuation">(</span>nv<span class="token double-colon punctuation">::</span><span class="token function">mu_to_ns</span><span class="token punctuation">(</span><span class="token function">get_mu_highest</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">+</span> l2a_allowed_latency_<span class="token punctuation">)</span><span class="token punctuation">)</span>
<span class="token punctuation">{</span>
    <span class="token function">NVLOGW_FMT</span><span class="token punctuation">(</span>TAG<span class="token punctuation">,</span> <span class="token string">"L2+L2A processing taking &gt; 500us ... Drop slot"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword keyword-return">return</span> <span class="token boolean">false</span><span class="token punctuation">;</span>   <span class="token comment">// 丢弃该 slot</span>
<span class="token punctuation">}</span>
</code></pre><p>其中 <code>l2a_allowed_latency_</code> 默认 <strong>100,000 ns (100µs)</strong>（<code>nv_phy_module.hpp:676</code>，<code>l2_adapter_config_F08.yaml:46</code>）。丢弃后 <code>scf_5g_fapi_phy.cpp:596</code> 向 L2 发出晚 slot 错误：</p>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// scf_5g_fapi_phy.cpp:596</span>
<span class="token function">send_error_indication_l1</span><span class="token punctuation">(</span>SCF_FAPI_SLOT_INDICATION<span class="token punctuation">,</span> SCF_ERROR_CODE_MSG_LATE_SLOT_ERR<span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">// err_code = 0x34</span>
</code></pre><p><strong>核心结论</strong>：8CC 把上述两类截止期上的内存访问压力放大约 <strong>8 倍</strong>（U-plane 流数按 <code>eAxC_id_pusch:[8,0]</code> 还要再 ×2），任何一处共享内存资源（SM、BAR1、HARQ 池、PCIe、nvIPC）出现排队，都会沿"GPU 持久核轮询超时 → PUSCH 流水线延迟 → L2A 晚 slot 丢弃"链路放大为可观测的超时。</p>
<h3 id="112-原因一order-kernel-持久线程块的-sm-争用最主要原因">11.2 原因一：Order Kernel 持久线程块的 SM 争用（最主要原因） </h3>
<p><strong>(1) 持久块数量随小区数线性翻倍</strong></p>
<p>Order Kernel 按 <strong><code>num_order_cells × 2</code></strong> 个常驻线程块启动（<code>order_cuda_kernels.cu:5517</code>）：</p>
<pre data-role="codeBlock" data-info="cuda" class="language-cuda cuda"><code>// order_cuda_kernels.cu:5516-5520
int cudaBlocks = (num_order_cells);            // 线程块数 = 小区数
int numThreads = (commViaCpu==true)?256:128;
// block 0 收包(receive)、block 1 处理(process)，故 ×2
order_kernel_doca_single&lt;&lt;&lt;cudaBlocks * 2, numThreads, 0, stream&gt;&gt;&gt;( ... );
</code></pre><table>
<thead>
<tr>
<th>场景</th>
<th><code>num_order_cells</code></th>
<th>持久线程块数 (×2)</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>1CC</td>
<td>1</td>
<td>2</td>
<td>1 收 + 1 处理</td>
</tr>
<tr>
<td>4CC</td>
<td>4</td>
<td>8</td>
<td>—</td>
</tr>
<tr>
<td><strong>8CC</strong></td>
<td><strong>8</strong></td>
<td><strong>16</strong></td>
<td><strong>16 个常驻块持续自旋占用 SM</strong></td>
</tr>
</tbody>
</table>
<p>每个持久块在整 slot 周期内 <strong>自旋轮询接收内存</strong>（GDR 路径轮询 <strong>GPU VRAM</strong>，CPU 路径轮询 <strong>CPU Pinned</strong>），属于"忙等"型 SM 占用——它们在等待报文期间并不会让出 SM 给后续 cuPHY 计算核。</p>
<p><strong>(2) 与 MPS SM 配额叠加产生争用</strong></p>
<p>F08 配置为各信道划分了固定 MPS SM 配额（<code>cuphycontroller_F08.yaml:38-44</code>）：</p>
<table>
<thead>
<tr>
<th>参数</th>
<th>值</th>
<th>含义</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>mps_sm_pusch</code></td>
<td>108</td>
<td>PUSCH 上行 context SM 配额</td>
</tr>
<tr>
<td><code>mps_sm_pucch</code></td>
<td>16</td>
<td>PUCCH</td>
</tr>
<tr>
<td><code>mps_sm_prach</code></td>
<td>2</td>
<td>PRACH</td>
</tr>
<tr>
<td><code>mps_sm_pdsch</code></td>
<td>82</td>
<td>PDSCH 下行</td>
</tr>
<tr>
<td><code>mps_sm_pdcch</code></td>
<td>36</td>
<td>PDCCH</td>
</tr>
<tr>
<td><code>mps_sm_pbch</code></td>
<td>36</td>
<td>PBCH/SSB</td>
</tr>
<tr>
<td><code>mps_sm_srs</code></td>
<td>16</td>
<td>SRS</td>
</tr>
</tbody>
</table>
<p>A100 共 108 个 SM。PUSCH context 仅分得 <code>mps_sm_pusch=108</code> 的逻辑配额（MPS 按线程百分比切分），而 8CC 的 <strong>16 个 Order 持久块</strong> 必须先在这块 SM 配额里"卡位"自旋，剩余 SM 才能跑 PUSCH 信道估计/均衡/LDPC。当 16 个持久块把 PUSCH SM 分区占满时：</p>
<ul>
<li>后续 cuPHY 计算核排队 → 单 slot 处理被拉长 → 触达 <code>check_time_threshold()</code> 的 <strong>500µs</strong> 截止期 → <strong>晚 slot 丢弃</strong>；</li>
<li>若收包本身因 RU 抖动稍晚，16 个块同时逼近 <code>ul_order_timeout_first_pkt_gpu_ns=1.15ms</code> / <code>ul_order_timeout_gpu_ns=2.3ms</code>，集中触发 <code>ORDER_KERNEL_EXIT_TIMEOUT_*</code>。</li>
</ul>
<p><strong>(3) 为什么 8CC 比低载更易超时</strong></p>
<ul>
<li>持久块是"先占 SM 再轮询"，块数翻倍即 SM 占用翻倍，<strong>收包等待与计算抢占同一 SM 池</strong>；</li>
<li><code>pusch_aggr_per_ctx=3</code>、<code>srs_aggr_per_ctx=3</code>、<code>pucch_aggr_per_ctx=4</code>（<code>F08.yaml:114-117</code>）意味着多个 aggr 对象共享同一 CUcontext 的 SM 配额，8CC 下并发 aggr 实例增多，进一步挤压 Order 持久块可用的 SM。</li>
</ul>
<blockquote>
<p><strong>优化方向</strong>：将 Order Kernel 收包块与处理块分离到独立的低配额 MPS context（或 Green Context），避免与 PUSCH 计算核争用同一 SM 分区；或在 8CC 下改用事件驱动/中断式收包替代 16 块忙等自旋。</p>
</blockquote>
<h3 id="113-原因二gdr-bar1-窗口压力与隐式-pcie-回退">11.3 原因二：GDR BAR1 窗口压力与隐式 PCIe 回退 </h3>
<p>F08 启用 GDR 零拷贝（<code>gpu_init_comms_dl: 1</code>），NIC 通过 GPUDirect RDMA 把 U-plane 报文 DMA 直写 <strong>GPU VRAM</strong>，该 VRAM 需经 <strong>BAR1 窗口</strong> 映射给 NIC 访问（见第 10 章 10.4 带宽参考）。</p>
<table>
<thead>
<tr>
<th>资源</th>
<th>说明</th>
<th>8CC 影响</th>
</tr>
</thead>
<tbody>
<tr>
<td>BAR1 窗口</td>
<td>A100 默认 64 GiB，所有 <code>gpinned_buffer</code>/GDR 缓冲共享</td>
<td>8 小区各自的 <code>ULInputBuffer</code>（<code>ul_input_buffer_per_cell=10</code>）+ SRS 缓冲（<code>ul_input_buffer_per_cell_srs=6</code>）+ PRACH 缓冲全部映射进同一 BAR1</td>
</tr>
<tr>
<td>单小区 UL 缓冲</td>
<td>10 个输入缓冲 × 8 小区 = <strong>80 个</strong> GDR 接收缓冲</td>
<td>数量随小区线性增长，逼近 BAR1 上限</td>
</tr>
<tr>
<td>SRS 缓冲</td>
<td>6 × 8 = 48 个</td>
<td>与 PUSCH 缓冲叠加</td>
</tr>
</tbody>
</table>
<p><strong>超时机理</strong>：</p>
<ol>
<li><strong>BAR1 超限静默回退</strong>——当所有 <code>gpinned_buffer</code> 累计映射超过 BAR1 窗口，<code>gdr_pin_buffer</code> 会静默回退到 <strong>CPU Pinned</strong>（见 10.6 表第 8 行）。一旦回退，NIC 报文先落 <strong>CPU Pinned</strong>，再经 <strong>PCIe H2D</strong> 拷入 <strong>GPU VRAM</strong>，Order Kernel 轮询的目标内存被插入一跳额外拷贝，单 slot 收齐时间变长 → 触发 <code>ORDER_KERNEL_EXIT_TIMEOUT_RX_PKT</code>。</li>
<li><strong>BAR1 带宽争用</strong>——即使未超限，8 小区报文并发 DMA 经同一 BAR1 窗口（实测 50–100 GB/s，远低于 HBM2e 的 2039 GB/s），高 PRB 占用 slot 下 BAR1 成为瓶颈，报文到达 GPU VRAM 的时刻整体后移。</li>
</ol>
<blockquote>
<p><strong>优化方向</strong>：提前统计全部 <code>gpinned_buffer</code> 总量并与 BAR1 上限做静态校验；监控 <code>gdr_pin_buffer</code> 返回码，超限显式告警而非静默回退；8CC 下按需缩减 <code>ul_input_buffer_per_cell</code>。</p>
</blockquote>
<h3 id="114-原因三共享-harq-池与-ul-输入缓冲耗尽">11.4 原因三：共享 HARQ 池与 UL 输入缓冲耗尽 </h3>
<p><strong>(1) HARQ 合并缓冲池全局共享（GPU VRAM）</strong></p>
<p><code>max_harq_pools</code> 默认 <strong>384</strong>（<code>yamlparser.cpp:1320-1321</code>），这是 <strong>所有小区、所有 UE 共享</strong> 的 HARQ 合并缓冲池总数（缓冲分配在 <strong>GPU VRAM</strong>，见第 10 章 HARQ Pool 行，单 buffer 约 7.6 MiB）。<code>harq_pool.cpp:364</code> 在池占用率低于 30% 时才回收：</p>
<pre data-role="codeBlock" data-info="cpp" class="language-cpp cpp"><code><span class="token comment">// harq_pool.cpp:364</span>
<span class="token keyword keyword-if">if</span><span class="token punctuation">(</span>hb_pool_list<span class="token punctuation">[</span>MAX_HARQ_POOLS<span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token operator">-&gt;</span><span class="token function">countElements</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token operator">*</span><span class="token number">100</span><span class="token operator">/</span><span class="token punctuation">(</span>pdctx<span class="token operator">-&gt;</span><span class="token function">getMaxHarqPools</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">&lt;</span> <span class="token number">30</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span> <span class="token punctuation">}</span>
</code></pre><p>8CC 下每小区可调度的 UE/HARQ 进程数叠加，384 个池可能不足以覆盖 8 小区 × 多 UE × 多 HARQ 进程的峰值并发。当池耗尽：</p>
<ul>
<li>新到 TB 申请不到 HARQ 缓冲（<strong>GPU VRAM</strong>）→ PUSCH 解码流水线阻塞等待，单 slot 处理超 500µs → <strong>晚 slot 丢弃</strong>；</li>
<li>上一 slot 的 HARQ 缓冲尚未释放（<code>notify_ul_harq_buffer_release: 0</code> 表示未启用主动释放通知），D2H 回读未及时完成，加剧 <strong>GPU VRAM</strong> 占用。</li>
</ul>
<p><strong>(2) UL 输入缓冲环按小区数倍增</strong></p>
<p><code>ul_input_buffer_per_cell=10</code>（<code>F08.yaml:118</code>）是 <strong>每小区</strong> 的 PUSCH 输入缓冲环深度。8CC 总计 80 个输入缓冲（GDR 路径在 <strong>GPU VRAM</strong>，CPU 路径在 <strong>CPU Pinned</strong>）。若某小区报文到达突发、Order Kernel 处理变慢，10 深的环被填满后新报文 <strong>无缓冲可落</strong> → NIC 侧丢包 → Order Kernel 永远等不到完整 slot 的所有 PRB → <code>ORDER_KERNEL_EXIT_TIMEOUT_RX_PKT</code>。</p>
<table>
<thead>
<tr>
<th>配置</th>
<th>值</th>
<th>8CC 总量</th>
<th>风险</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ul_input_buffer_per_cell</code></td>
<td>10</td>
<td>10×8 = 80</td>
<td>环满即丢包，触发收包超时</td>
</tr>
<tr>
<td><code>ul_input_buffer_per_cell_srs</code></td>
<td>6</td>
<td>6×8 = 48</td>
<td>SRS 突发下不足</td>
</tr>
<tr>
<td><code>max_harq_pools</code></td>
<td>384</td>
<td>全局共享</td>
<td>多 UE 峰值耗尽 → 解码阻塞</td>
</tr>
</tbody>
</table>
<blockquote>
<p><strong>优化方向</strong>：8CC 按实际 UE 数上调 <code>max_harq_pools</code>；启用 <code>notify_ul_harq_buffer_release</code> 以尽早回收 <strong>GPU VRAM</strong>；按小区 PRB 峰值适当增大 <code>ul_input_buffer_per_cell</code>（同时核对 BAR1 余量，避免触发 11.3 的回退）。</p>
</blockquote>
<h3 id="115-原因四pcie-h2dd2h-带宽饱和">11.5 原因四：PCIe H2D/D2H 带宽饱和 </h3>
<p>第 10 章 10.1/10.2 指出的所有跨 PCIe 搬运在 8CC 下并发 8 倍，PCIe 5.0 x16 实测约 128 GB/s 为全系统所有小区共享：</p>
<table>
<thead>
<tr>
<th>跨 PCIe 内存搬运</th>
<th>方向</th>
<th>内存类型</th>
<th>8CC 并发度</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ULInputBuffer</code> H2D（无 GDR 时）</td>
<td>CPU→GPU</td>
<td><strong>CPU Pinned → GPU VRAM</strong></td>
<td>×8 小区</td>
</tr>
<tr>
<td>TB CRC / UCI 结果 D2H</td>
<td>GPU→CPU</td>
<td><strong>GPU VRAM → CPU Pinned</strong></td>
<td>×8 小区</td>
</tr>
<tr>
<td>DL TB 数据 H2D</td>
<td>CPU→GPU</td>
<td><strong>CPU Pinned → GPU VRAM</strong></td>
<td>×8 小区</td>
</tr>
<tr>
<td>BFW 权重下发</td>
<td>CPU→GPU</td>
<td><strong>CPU Pinned → GPU VRAM</strong></td>
<td>×8 小区</td>
</tr>
<tr>
<td>cuMAC 调度结果 D2H 镜像</td>
<td>GPU→CPU</td>
<td><strong>GPU VRAM → CPU Pinned</strong></td>
<td>每 slot 全量</td>
</tr>
</tbody>
</table>
<p><strong>超时机理</strong>：UL 与 DL 在相邻 slot 交错，8 小区的 H2D（DL 数据 + BFW）与 D2H（UL CRC/UCI + cuMAC 镜像）在同一 PCIe 链路上排队。当某 slot 多个大块传输叠加（如 DL 大 TB + 全量 BFW），PCIe 队列延迟把数据送达 <strong>GPU VRAM</strong> 的时刻推后，使 GPU 计算核启动延迟，最终 L2A 处理超 500µs。<code>workers_ul:[2,3]</code> / <code>workers_dl:[4,5,6]</code>（<code>F08.yaml:68-74</code>）的 DMA 提交线程有限，进一步限制并发拷贝引擎利用率。</p>
<blockquote>
<p><strong>优化方向</strong>：尽量走 GDR 让 NIC 直达 <strong>GPU VRAM</strong> 绕过 H2D；D2H 仅回传 dirty 字段（见 10.6 第 5 行）；利用多 copy engine 并按 UL/DL 分离 stream 错峰提交。</p>
</blockquote>
<h3 id="116-原因五nvipc-共享池与-cpu-pinned-争用">11.6 原因五：nvIPC 共享池与 CPU Pinned 争用 </h3>
<p>L2↔L1 经 nvIPC 共享内存交互（第 10 章 10.5 拓扑）。<code>cpu_data</code> 池为 <strong>CPU Pinned</strong>（约 564 MiB 常驻，见 10.6 第 4 行），由所有小区的 FAPI 消息共享。</p>
<p><strong>超时机理</strong>：</p>
<ol>
<li>8CC 下 8 小区的 SLOT/TX_DATA/RX_DATA/CRC/UCI 等 FAPI 消息并发申请同一 nvIPC <code>cpu_data</code> 池缓冲，池满时 <code>tx_alloc()</code> 失败（见 <code>scf_5g_fapi_phy.cpp:606</code> 的 <code>tx_alloc(msg_desc) &lt; 0</code> 分支）→ 消息延迟或丢弃。</li>
<li>大量 <strong>CPU Pinned</strong> 常驻给 CPU TLB/LLC 带来压力，FAPI 解析与拷贝变慢，叠加进 L2A 的 100µs <code>l2a_allowed_latency_</code> 预算，更易越过 500µs 截止期。</li>
<li>nvIPC 唤醒（event/sem）在 8 小区高频触发下产生 CPU 调度抖动，延后 GPU 任务提交。</li>
</ol>
<blockquote>
<p><strong>优化方向</strong>：评估对热路径消息改用 <code>cuda_data</code>（<strong>GPU VRAM</strong>）全程 GPU IPC，跳过 CPU Pinned 中转（见 10.6 第 4 行）；按 8CC 消息峰值扩容 nvIPC 池并绑定独立 CPU 核降低调度抖动。</p>
</blockquote>
<h3 id="117-原因六cumac-逐字段分配与-d2h-镜像随小区放大">11.7 原因六：cuMAC 逐字段分配与 D2H 镜像随小区放大 </h3>
<p>第 10 章指出 cuMAC <code>cumacSubcontext</code> 采用 <strong>逐字段 <code>cudaMalloc</code></strong>（<strong>GPU VRAM</strong>，见 10.6 第 2 行），并每次调度做 GPU→CPU 全量镜像 D2H（<strong>GPU VRAM → CPU Pinned</strong>，10.6 第 5 行）。</p>
<p><strong>超时机理</strong>：</p>
<ol>
<li>8 小区的调度结构体字段数倍增，逐字段 <code>cudaMalloc</code> 在运行期若涉及扩容会触发 CUDA driver 全局锁，阻塞其它 stream 的内存操作（包括 Order Kernel 依赖的缓冲准备）。</li>
<li>每 slot 全量镜像 D2H 把 8 小区未变更字段也一并回拷，占用 PCIe（叠加 11.5）并延后 CPU 侧读取调度结果，使 DL 准备链路变慢。</li>
</ol>
<blockquote>
<p><strong>优化方向</strong>：单次大块 <code>cudaMalloc</code> + 偏移切分替代逐字段分配（10.6 第 2 行）；改用 <code>cudaHostAllocMapped</code>（<strong>CPU Pinned Mapped</strong>）共享调度结果或仅增量同步 dirty 字段，消除每 slot 全量 D2H。</p>
</blockquote>
<h3 id="118-原因归纳与内存类型映射">11.8 原因归纳与内存类型映射 </h3>
<p>下表把 8CC 超时的各根因映射到第 10 章的内存类型，并标注超时表现形式与放大系数：</p>
<table>
<thead>
<tr>
<th>#</th>
<th>根因</th>
<th>涉及内存类型</th>
<th>8CC 放大</th>
<th>超时表现</th>
<th>源码/配置锚点</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Order Kernel 持久块 SM 争用</td>
<td><strong>GPU VRAM</strong>(轮询)/SM 分区</td>
<td>2→16 块 (×8)</td>
<td><code>ORDER_KERNEL_EXIT_TIMEOUT_*</code> + 晚 slot</td>
<td><code>order_cuda_kernels.cu:5517</code>，<code>mps_sm_pusch=108</code></td>
</tr>
<tr>
<td>2</td>
<td>GDR BAR1 窗口超限/回退</td>
<td><strong>GPU VRAM ↔ BAR1</strong>，回退 <strong>CPU Pinned</strong></td>
<td>80+48 缓冲</td>
<td>隐式 PCIe 拷贝→收包超时</td>
<td>10.6 第 8 行，<code>gpu_init_comms_dl=1</code></td>
</tr>
<tr>
<td>3</td>
<td>HARQ 池耗尽</td>
<td><strong>GPU VRAM</strong> (7.6 MiB×N)</td>
<td>384 全局共享</td>
<td>解码阻塞→晚 slot</td>
<td><code>max_harq_pools=384</code>，<code>harq_pool.cpp:364</code></td>
</tr>
<tr>
<td>4</td>
<td>UL 输入缓冲环满</td>
<td><strong>GPU VRAM</strong>/<strong>CPU Pinned</strong></td>
<td>10×8=80</td>
<td>丢包→<code>TIMEOUT_RX_PKT</code></td>
<td><code>ul_input_buffer_per_cell=10</code></td>
</tr>
<tr>
<td>5</td>
<td>PCIe H2D/D2H 饱和</td>
<td><strong>CPU Pinned ↔ GPU VRAM</strong></td>
<td>×8 小区并发</td>
<td>计算核延迟→晚 slot</td>
<td>10.1/10.2，PCIe 128 GB/s</td>
</tr>
<tr>
<td>6</td>
<td>nvIPC 池争用</td>
<td><strong>CPU Pinned</strong> (~564 MiB)</td>
<td>×8 消息并发</td>
<td><code>tx_alloc</code> 失败/抖动</td>
<td>10.6 第 4 行，<code>scf_5g_fapi_phy.cpp:606</code></td>
</tr>
<tr>
<td>7</td>
<td>cuMAC 逐字段分配+全量 D2H</td>
<td><strong>GPU VRAM → CPU Pinned</strong></td>
<td>字段数×8</td>
<td>driver 锁/PCIe→DL 延迟</td>
<td>10.6 第 2/5 行</td>
</tr>
</tbody>
</table>
<p><strong>触发链路总览</strong>（任一根因都可经此链路放大为超时）：</p>
<pre data-role="codeBlock" data-info="" class="language-text"><code>8 小区 U-plane 报文并发
        │
        ├─[原因1] 16 个 Order 持久块抢占 PUSCH SM 分区 ─┐
        ├─[原因2] BAR1 超限→静默回退 CPU Pinned+PCIe 跳 ─┤
        ├─[原因3] HARQ 池(384) 耗尽→解码阻塞 ───────────┤
        ├─[原因4] UL 输入环(10/cell) 满→NIC 丢包 ───────┼─→ GPU 收包/计算延迟
        ├─[原因5] PCIe(128GB/s) H2D/D2H 排队 ───────────┤
        ├─[原因6] nvIPC cpu_data 池满+CPU 抖动 ─────────┤
        └─[原因7] cuMAC driver 锁+全量 D2H ─────────────┘
        │
        ▼
   GPU 持久核轮询接收内存超时
   (ul_order_timeout_first_pkt=1.15ms / gpu=2.3ms)
        │  ORDER_KERNEL_EXIT_TIMEOUT_RX_PKT / NO_PKT
        ▼
   PUSCH 流水线被拉长 / 终止 (task_function_ul_aggr.cpp:2404)
        │
        ▼
   L2+L2A 处理 &gt; 500µs  (check_time_threshold, l2a_allowed_latency=100µs)
        │
        ▼
   Drop slot → FAPI SCF_ERROR_CODE_MSG_LATE_SLOT_ERR (0x34)
   (scf_5g_fapi_phy.cpp:596)
</code></pre><h3 id="119-定位与排查清单">11.9 定位与排查清单 </h3>
<p>按内存子系统逐项核对，快速定位 8CC 超时的主导因素：</p>
<table>
<thead>
<tr>
<th>检查项</th>
<th>观测点</th>
<th>判定</th>
</tr>
</thead>
<tbody>
<tr>
<td>Order Kernel 超时计数</td>
<td>日志 <code>Order kernel timeout error (exit condition ...)</code></td>
<td><code>RX_PKT</code> 多→收包/缓冲问题；<code>NO_PKT</code> 多→FH/SM 卡位</td>
</tr>
<tr>
<td>晚 slot 计数</td>
<td>日志 <code>L2+L2A processing taking &gt; 500us ... Drop slot</code></td>
<td>频繁→整链路延迟（SM/PCIe/HARQ）</td>
</tr>
<tr>
<td>FAPI 错误码</td>
<td>L2 收到 <code>0x34 LATE_SLOT_ERR</code></td>
<td>确认晚 slot 已上报</td>
</tr>
<tr>
<td>SM 占用</td>
<td>Nsight Systems 观察 PUSCH context SM 利用</td>
<td>16 持久块是否占满 <code>mps_sm_pusch</code> 分区</td>
</tr>
<tr>
<td>BAR1 用量</td>
<td><code>nvidia-smi -q</code> BAR1 Memory；<code>gdr_pin_buffer</code> 返回码</td>
<td>是否逼近 64 GiB / 是否回退</td>
</tr>
<tr>
<td>HARQ 池占用</td>
<td><code>harq_pool</code> 计数 vs <code>max_harq_pools=384</code></td>
<td>是否接近耗尽</td>
</tr>
<tr>
<td>PCIe 带宽</td>
<td>Nsight 观察 H2D/D2H 吞吐 vs 128 GB/s</td>
<td>是否饱和</td>
</tr>
<tr>
<td>nvIPC 池</td>
<td><code>tx_alloc</code> 失败日志</td>
<td>池是否打满</td>
</tr>
</tbody>
</table>
<blockquote>
<p><strong>8CC 调优优先级建议</strong>：先解决 <strong>原因 1（SM 争用）</strong> 与 <strong>原因 2（BAR1/GDR）</strong>——二者直接决定 GPU 持久核能否按时拿到接收内存；再处理 <strong>原因 3/4（HARQ/输入缓冲）</strong> 的容量配置；最后优化 <strong>原因 5/6/7</strong> 的 PCIe 与 CPU 侧搬运。所有改动需同时校验 BAR1 与 GPU VRAM 余量，避免顾此失彼。</p>
</blockquote>

      </div>
      
      
    
    
    
    
    
    
  
    </body></html>
