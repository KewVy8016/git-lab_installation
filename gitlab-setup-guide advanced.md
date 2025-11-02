---


---

<h1 id="คู่มือการติดตั้ง-gitlab-ce-และ-gitlab-runner-บน-docker">คู่มือการติดตั้ง GitLab CE และ GitLab Runner บน Docker</h1>
<h2 id="บทนำ">บทนำ</h2>
<p>เอกสารนี้จะอธิบายขั้นตอนการติดตั้ง GitLab Community Edition (CE) พร้อมกับ GitLab Runner โดยใช้ Docker Compose สำหรับการทำ CI/CD (Continuous Integration/Continuous Deployment) บนระบบปฏิบัติการ Ubuntu Server</p>
<h3 id="ความเข้าใจเกี่ยวกับ-gitlab-และ-gitlab-runner">ความเข้าใจเกี่ยวกับ GitLab และ GitLab Runner</h3>
<p><strong>GitLab CE (Community Edition)</strong> คือระบบบริหารจัดการ Source Code แบบ Open Source ที่มีฟีเจอร์ครบครันสำหรับการทำ DevOps รวมถึง:</p>
<ul>
<li>Version Control (Git)</li>
<li>Issue Tracking</li>
<li>Code Review</li>
<li>CI/CD Pipeline</li>
<li>Container Registry</li>
</ul>
<p><strong>GitLab Runner</strong> คือโปรแกรมที่ทำหน้าที่รัน CI/CD Jobs ที่ถูกกำหนดใน <code>.gitlab-ci.yml</code> โดย Runner จะรับคำสั่งจาก GitLab Server และดำเนินการตามที่กำหนด</p>
<hr>
<h2 id="ข้อกำหนดเบื้องต้น">ข้อกำหนดเบื้องต้น</h2>
<p>ก่อนเริ่มการติดตั้ง กรุณาตรวจสอบให้แน่ใจว่าระบบของคุณมีส่วนประกอบต่อไปนี้:</p>
<h3 id="ข้อกำหนดด้านซอฟต์แวร์">ข้อกำหนดด้านซอฟต์แวร์</h3>
<ul>
<li><strong>Docker Engine</strong> เวอร์ชัน 20.10 ขึ้นไป</li>
<li><strong>Docker Compose</strong> เวอร์ชัน 1.29 ขึ้นไป (หรือ Docker Compose V2)</li>
<li><strong>Ubuntu Server</strong> แนะนำ 20.04 LTS ขึ้นไป</li>
<li>สิทธิ์ผู้ดูแลระบบ (sudo)</li>
</ul>
<h3 id="ข้อกำหนดด้านฮาร์ดแวร์">ข้อกำหนดด้านฮาร์ดแวร์</h3>
<ul>
<li><strong>CPU:</strong> อย่างน้อย 4 cores</li>
<li><strong>RAM:</strong> อย่างน้อย 8 GB (แนะนำ 16 GB)</li>
<li><strong>Storage:</strong> อย่างน้อย 50 GB ของพื้นที่ว่าง</li>
<li><strong>Network:</strong> การเชื่อมต่ออินเทอร์เน็ตสำหรับดาวน์โหลด Docker Images</li>
</ul>
<hr>
<h2 id="ขั้นตอนที่-1-การเตรียมโครงสร้างไดเรกทอรี">ขั้นตอนที่ 1: การเตรียมโครงสร้างไดเรกทอรี</h2>
<h3 id="สร้างไดเรกทอรีโครงการ">1.1 สร้างไดเรกทอรีโครงการ</h3>
<p>สร้างไดเรกทอรีหลักสำหรับโครงการและเข้าสู่ไดเรกทอรีนั้น:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">mkdir</span> gitlab-ci-cd
<span class="token function">cd</span> gitlab-ci-cd
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li>คำสั่ง <code>mkdir</code> จะสร้างไดเรกทอรีใหม่ชื่อ <code>gitlab-ci-cd</code></li>
<li>คำสั่ง <code>cd</code> จะเปลี่ยนตำแหน่งปัจจุบันเข้าไปในไดเรกทอรีที่สร้างขึ้น</li>
</ul>
<h3 id="สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล">1.2 สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล</h3>
<p>สร้างโครงสร้างไดเรกทอรีย่อยสำหรับ Persistent Storage:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">mkdir</span> -p gitlab/config gitlab/logs gitlab/data
<span class="token function">mkdir</span> -p gitlab-runner/config
</code></pre>
<p><strong>คำอธิบายโครงสร้างไดเรกทอรี:</strong></p>
<pre><code>gitlab-ci-cd/
├── gitlab/
│   ├── config/      # เก็บไฟล์การตั้งค่า GitLab (gitlab.rb, ssl certificates)
│   ├── logs/        # เก็บ log files ทั้งหมดของ GitLab
│   └── data/        # เก็บข้อมูลหลัก (repositories, database, uploads)
└── gitlab-runner/
    └── config/      # เก็บการตั้งค่าของ Runner (config.toml)
</code></pre>
<p><strong>ความสำคัญของ Persistent Volumes:</strong></p>
<ul>
<li>ข้อมูลจะไม่สูญหายเมื่อ Container ถูกลบหรือรีสตาร์ท</li>
<li>สามารถ backup และ restore ได้ง่าย</li>
<li>สามารถอัพเกรด GitLab โดยไม่สูญเสียข้อมูล</li>
</ul>
<p><strong>หมายเหตุ:</strong> ตัวเลือก <code>-p</code> ใน mkdir จะสร้างไดเรกทอรีแบบ nested โดยอัตโนมัติ</p>
<hr>
<h2 id="ขั้นตอนที่-2-การสร้างไฟล์-docker-compose">ขั้นตอนที่ 2: การสร้างไฟล์ Docker Compose</h2>
<p>สร้างไฟล์ <code>docker-compose.yml</code> ในไดเรกทอรี <code>gitlab-ci-cd</code> ด้วยเนื้อหาดังนี้:</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token comment"># กำหนดเวอร์ชันของ Docker Compose File Format</span>
<span class="token comment"># เวอร์ชัน 3.7 รองรับฟีเจอร์ครบถ้วนและเสถียร</span>
<span class="token key atrule">version</span><span class="token punctuation">:</span> <span class="token string">"3.7"</span>

<span class="token comment"># กำหนด Services ที่จะรันใน Docker Environment</span>
<span class="token key atrule">services</span><span class="token punctuation">:</span>
  
  <span class="token comment"># ===================================</span>
  <span class="token comment"># Service: GitLab Server</span>
  <span class="token comment"># ===================================</span>
  <span class="token key atrule">gitlab</span><span class="token punctuation">:</span>
    <span class="token comment"># ใช้ Official GitLab CE Image เวอร์ชันล่าสุดจาก Docker Hub</span>
    <span class="token key atrule">image</span><span class="token punctuation">:</span> <span class="token string">'gitlab/gitlab-ce:latest'</span>
    
    <span class="token comment"># ตั้งชื่อ Container เพื่อง่ายต่อการจัดการ</span>
    <span class="token key atrule">container_name</span><span class="token punctuation">:</span> gitlab
    
    <span class="token comment"># กำหนดให้ Container รีสตาร์ทอัตโนมัติเมื่อหยุดทำงานหรือเซิร์ฟเวอร์รีบูต</span>
    <span class="token key atrule">restart</span><span class="token punctuation">:</span> always
    
    <span class="token comment"># กำหนด hostname ของ Container (ควรตรงกับ external_url)</span>
    <span class="token comment"># ใช้สำหรับการ resolve DNS ภายใน Docker network</span>
    <span class="token key atrule">hostname</span><span class="token punctuation">:</span> <span class="token string">'gitlab.example.com'</span>
    
    <span class="token comment"># กำหนดตัวแปร Environment Variables</span>
    <span class="token key atrule">environment</span><span class="token punctuation">:</span>
      <span class="token comment"># GITLAB_OMNIBUS_CONFIG: ใช้สำหรับกำหนดค่า GitLab แบบ inline</span>
      <span class="token comment"># เขียนในรูปแบบ Ruby configuration</span>
      <span class="token key atrule">GITLAB_OMNIBUS_CONFIG</span><span class="token punctuation">:</span> <span class="token punctuation">|</span><span class="token scalar string">
        # external_url: URL ที่ผู้ใช้จะใช้เข้าถึง GitLab
        # ต้องเปลี่ยนเป็น IP Address หรือ Domain Name จริงของเซิร์ฟเวอร์
        external_url 'http://gitlab.example.com'</span>
        
        <span class="token comment"># (ตัวเลือก) หากพอร์ต SSH (22) ถูกใช้งานโดย Host อยู่แล้ว</span>
        <span class="token comment"># ให้ uncomment บรรทัดด้านล่างและเปลี่ยน Port mapping เป็น '2222:22'</span>
        <span class="token comment"># gitlab_rails['gitlab_shell_ssh_port'] = 2222</span>
        
        <span class="token comment"># (ตัวเลือก) การตั้งค่า Email notification</span>
        <span class="token comment"># gitlab_rails['smtp_enable'] = true</span>
        <span class="token comment"># gitlab_rails['smtp_address'] = "smtp.gmail.com"</span>
        <span class="token comment"># gitlab_rails['smtp_port'] = 587</span>
        
        <span class="token comment"># (ตัวเลือก) การตั้งค่า Backup</span>
        <span class="token comment"># gitlab_rails['backup_keep_time'] = 604800</span>
    
    <span class="token comment"># Port Mapping: แมพพอร์ตของ Host กับพอร์ตของ Container</span>
    <span class="token comment"># รูปแบบ: 'HOST_PORT:CONTAINER_PORT'</span>
    <span class="token key atrule">ports</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> '80<span class="token punctuation">:</span>80'     <span class="token comment"># HTTP - ใช้สำหรับเข้าถึง Web Interface</span>
      <span class="token punctuation">-</span> '443<span class="token punctuation">:</span>443'   <span class="token comment"># HTTPS - ใช้สำหรับเข้าถึงแบบ Secure (ต้องมี SSL Certificate)</span>
      <span class="token punctuation">-</span> '22<span class="token punctuation">:</span>22'     <span class="token comment"># SSH - ใช้สำหรับ Git operations (clone, push, pull)</span>
                    <span class="token comment"># หากชนกับ SSH ของ Host ให้เปลี่ยนเป็น '2222:22'</span>
    
    <span class="token comment"># Volume Mapping: แมพไดเรกทอรีของ Host กับไดเรกทอรีของ Container</span>
    <span class="token comment"># รูปแบบ: 'HOST_PATH:CONTAINER_PATH'</span>
    <span class="token key atrule">volumes</span><span class="token punctuation">:</span>
      <span class="token comment"># เก็บไฟล์การตั้งค่าทั้งหมด เช่น gitlab.rb, ssl certificates</span>
      <span class="token punctuation">-</span> <span class="token string">'./gitlab/config:/etc/gitlab'</span>
      
      <span class="token comment"># เก็บ log files ทั้งหมด เพื่อการ troubleshooting</span>
      <span class="token punctuation">-</span> <span class="token string">'./gitlab/logs:/var/log/gitlab'</span>
      
      <span class="token comment"># เก็บข้อมูลหลักทั้งหมด เช่น:</span>
      <span class="token comment"># - Git repositories</span>
      <span class="token comment"># - PostgreSQL database</span>
      <span class="token comment"># - Redis data</span>
      <span class="token comment"># - Uploaded files</span>
      <span class="token comment"># - Container Registry images</span>
      <span class="token punctuation">-</span> <span class="token string">'./gitlab/data:/var/opt/gitlab'</span>
    
    <span class="token comment"># Shared Memory Size: กำหนดขนาด shared memory</span>
    <span class="token comment"># GitLab ใช้ /dev/shm สำหรับ PostgreSQL และ Redis</span>
    <span class="token comment"># ค่าต่ำเกินไปอาจทำให้เกิดปัญหาด้านประสิทธิภาพ</span>
    <span class="token key atrule">shm_size</span><span class="token punctuation">:</span> <span class="token string">'256m'</span>
    
    <span class="token comment"># (ตัวเลือก) Networks: กำหนด network ที่ Container จะเชื่อมต่อ</span>
    <span class="token comment"># หากไม่ระบุ Docker จะสร้าง default network ให้อัตโนมัติ</span>
    <span class="token comment"># networks:</span>
    <span class="token comment">#   - gitlab-network</span>
  
  <span class="token comment"># ===================================</span>
  <span class="token comment"># Service: GitLab Runner</span>
  <span class="token comment"># ===================================</span>
  <span class="token key atrule">gitlab-runner</span><span class="token punctuation">:</span>
    <span class="token comment"># ใช้ Official GitLab Runner Image เวอร์ชันล่าสุด</span>
    <span class="token key atrule">image</span><span class="token punctuation">:</span> gitlab/gitlab<span class="token punctuation">-</span>runner<span class="token punctuation">:</span>latest
    
    <span class="token comment"># ตั้งชื่อ Container</span>
    <span class="token key atrule">container_name</span><span class="token punctuation">:</span> gitlab<span class="token punctuation">-</span>runner
    
    <span class="token comment"># รีสตาร์ทอัตโนมัติ</span>
    <span class="token key atrule">restart</span><span class="token punctuation">:</span> always
    
    <span class="token comment"># depends_on: รอให้ gitlab service เริ่มต้นก่อน</span>
    <span class="token comment"># (หมายเหตุ: นี่ไม่ได้รับประกันว่า GitLab จะพร้อมใช้งานแล้ว</span>
    <span class="token comment">#  แค่รับประกันว่า Container จะถูกสร้างตามลำดับ)</span>
    <span class="token key atrule">depends_on</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> gitlab
    
    <span class="token comment"># Volume Mapping สำหรับ Runner</span>
    <span class="token key atrule">volumes</span><span class="token punctuation">:</span>
      <span class="token comment"># เก็บไฟล์ config.toml ซึ่งมีข้อมูลการลงทะเบียนกับ GitLab</span>
      <span class="token punctuation">-</span> <span class="token string">'./gitlab-runner/config:/etc/gitlab-runner'</span>
      
      <span class="token comment"># Mount Docker Socket เพื่อให้ Runner สามารถสั่ง Docker daemon ของ Host</span>
      <span class="token comment"># วิธีนี้เรียกว่า "Docker-in-Docker" (DinD) แบบ Socket Binding</span>
      <span class="token comment"># ทำให้ Runner สามารถ:</span>
      <span class="token comment"># - สร้าง Docker containers สำหรับรัน CI/CD jobs</span>
      <span class="token comment"># - Build Docker images</span>
      <span class="token comment"># - Push/Pull images จาก Container Registry</span>
      <span class="token punctuation">-</span> <span class="token string">'/var/run/docker.sock:/var/run/docker.sock'</span>
    
    <span class="token comment"># (ตัวเลือก) Environment Variables สำหรับ Runner</span>
    <span class="token comment"># environment:</span>
    <span class="token comment">#   - DOCKER_HOST=unix:///var/run/docker.sock</span>
    
    <span class="token comment"># (ตัวเลือก) Privileged Mode: ให้ Container มีสิทธิ์เข้าถึงระดับสูง</span>
    <span class="token comment"># จำเป็นสำหรับบาง Executor เช่น Docker-in-Docker แบบ privileged</span>
    <span class="token comment"># privileged: true</span>

<span class="token comment"># (ตัวเลือก) กำหนด Networks แบบ custom</span>
<span class="token comment"># networks:</span>
<span class="token comment">#   gitlab-network:</span>
<span class="token comment">#     driver: bridge</span>

<span class="token comment"># (ตัวเลือก) กำหนด Volumes แบบ named volumes แทน bind mounts</span>
<span class="token comment"># volumes:</span>
<span class="token comment">#   gitlab-config:</span>
<span class="token comment">#   gitlab-logs:</span>
<span class="token comment">#   gitlab-data:</span>
<span class="token comment">#   gitlab-runner-config:</span>
</code></pre>
<h3 id="คำอธิบายเพิ่มเติมเกี่ยวกับการตั้งค่าสำคัญ">คำอธิบายเพิ่มเติมเกี่ยวกับการตั้งค่าสำคัญ</h3>
<h4 id="external-url">External URL</h4>
<p><code>external_url</code> เป็นการตั้งค่าที่สำคัญที่สุด เพราะ GitLab จะใช้ค่านี้ในการ:</p>
<ul>
<li>สร้าง URL สำหรับ Clone repositories</li>
<li>สร้างลิงก์ใน Email notifications</li>
<li>กำหนด Callback URLs สำหรับ OAuth</li>
<li>ตั้งค่า CORS policies</li>
</ul>
<p><strong>ตัวอย่างการตั้งค่า:</strong></p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token comment"># ใช้ IP Address</span>
external_url 'http<span class="token punctuation">:</span>//192.168.1.100'

<span class="token comment"># ใช้ Domain Name</span>
external_url 'http<span class="token punctuation">:</span>//gitlab.company.com'

<span class="token comment"># ใช้ HTTPS (ต้องมี SSL Certificate)</span>
external_url 'https<span class="token punctuation">:</span>//gitlab.company.com'
</code></pre>
<h4 id="port-mapping-strategy">Port Mapping Strategy</h4>
<p>มีสองกรณีหลักในการจัดการ ports:</p>
<p><strong>กรณีที่ 1: Host ไม่มีบริการอื่นทำงานที่พอร์ต 80, 443, 22</strong></p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">ports</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> <span class="token string">'80:80'</span>
  <span class="token punctuation">-</span> <span class="token string">'443:443'</span>
  <span class="token punctuation">-</span> <span class="token string">'22:22'</span>
</code></pre>
<p><strong>กรณีที่ 2: Host มีบริการอื่นใช้งานพอร์ตเหล่านี้อยู่แล้ว</strong></p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">ports</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> <span class="token string">'8080:80'</span>    <span class="token comment"># เข้าถึง GitLab ที่ http://server:8080</span>
  <span class="token punctuation">-</span> '8443<span class="token punctuation">:</span>443'   <span class="token comment"># เข้าถึง GitLab ที่ https://server:8443</span>
  <span class="token punctuation">-</span> '2222<span class="token punctuation">:</span>22'    <span class="token comment"># Git SSH ที่ port 2222</span>
  
<span class="token comment"># ต้องเพิ่มใน GITLAB_OMNIBUS_CONFIG</span>
<span class="token key atrule">environment</span><span class="token punctuation">:</span>
  <span class="token key atrule">GITLAB_OMNIBUS_CONFIG</span><span class="token punctuation">:</span> <span class="token punctuation">|</span><span class="token scalar string">
    external_url 'http://gitlab.example.com:8080'
    gitlab_rails['gitlab_shell_ssh_port'] = 2222</span>
</code></pre>
<h4 id="docker-socket-binding">Docker Socket Binding</h4>
<p>การ mount <code>/var/run/docker.sock</code> ทำให้ Runner สามารถควบคุม Docker daemon ของ Host ได้โดยตรง</p>
<p><strong>ข้อดี:</strong></p>
<ul>
<li>ง่ายต่อการตั้งค่า</li>
<li>ประสิทธิภาพดี (ไม่ต้องรัน nested Docker daemon)</li>
<li>ใช้ resources น้อยกว่า</li>
</ul>
<p><strong>ข้อควรระวัง:</strong></p>
<ul>
<li>Container มีสิทธิ์สูงในการควบคุม Docker</li>
<li>ควรใช้ในสภาพแวดล้อมที่เชื่อถือได้เท่านั้น</li>
<li>พิจารณาใช้ Docker-in-Docker แบบ privileged หากต้องการ isolation มากกว่า</li>
</ul>
<hr>
<h2 id="ขั้นตอนที่-3-การเริ่มระบบ">ขั้นตอนที่ 3: การเริ่มระบบ</h2>
<h3 id="เริ่มต้น-container">3.1 เริ่มต้น Container</h3>
<p>รันคำสั่งเพื่อเริ่มต้นระบบทั้งหมด:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker compose up -d
</code></pre>
<p><strong>คำอธิบายพารามิเตอร์:</strong></p>
<ul>
<li><code>docker compose</code>: คำสั่งสำหรับจัดการ multi-container applications</li>
<li><code>up</code>: สร้างและเริ่มต้น containers</li>
<li><code>-d</code> (detached mode): รัน containers ใน background</li>
</ul>
<p><strong>ขั้นตอนที่เกิดขึ้นเบื้องหลัง:</strong></p>
<ol>
<li>Docker จะดาวน์โหลด images (gitlab-ce และ gitlab-runner) ถ้ายังไม่มีใน local</li>
<li>สร้าง Docker network สำหรับให้ containers สื่อสารกัน</li>
<li>สร้าง volumes ตามที่กำหนด</li>
<li>สร้างและเริ่มต้น gitlab container</li>
<li>รอให้ gitlab container เริ่มต้นเสร็จ แล้วจึงสร้าง gitlab-runner container</li>
</ol>
<p><strong>ระยะเวลาที่คาดหวัง:</strong></p>
<ul>
<li>ดาวน์โหลด images: 5-10 นาที (ขึ้นอยู่กับความเร็วอินเทอร์เน็ต)</li>
<li>GitLab initialization: 5-15 นาที (ขึ้นอยู่กับประสิทธิภาพเซิร์ฟเวอร์)</li>
</ul>
<h3 id="ติดตามการเริ่มต้นระบบ">3.2 ติดตามการเริ่มต้นระบบ</h3>
<p>ดู real-time logs ของ GitLab:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker logs -f gitlab
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li><code>-f</code> (follow): แสดง logs แบบ real-time</li>
<li>กด <code>Ctrl+C</code> เพื่อออกจากการดู logs</li>
</ul>
<p><strong>ข้อความสำคัญที่ควรมองหา:</strong></p>
<pre><code>==&gt; /var/log/gitlab/reconfigure.stdout.log &lt;==
gitlab Reconfigured!
</code></pre>
<p>เมื่อเห็นข้อความนี้แสดงว่า GitLab พร้อมใช้งานแล้ว</p>
<h3 id="ตรวจสอบสถานะ">3.3 ตรวจสอบสถานะ</h3>
<p>ตรวจสอบว่า containers ทำงานปกติ:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">ps</span>
</code></pre>
<p><strong>ผลลัพธ์ที่คาดหวัง:</strong></p>
<pre><code>CONTAINER ID   IMAGE                         STATUS          PORTS                                       NAMES
abc123def456   gitlab/gitlab-ce:latest       Up 10 minutes   0.0.0.0:22-&gt;22, 0.0.0.0:80-&gt;80, 443-&gt;443   gitlab
xyz789ghi012   gitlab/gitlab-runner:latest   Up 9 minutes                                                gitlab-runner
</code></pre>
<p><strong>การตรวจสอบเพิ่มเติม:</strong></p>
<p>ตรวจสอบ resource usage:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker stats
</code></pre>
<p>ตรวจสอบ networks:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker network <span class="token function">ls</span>
</code></pre>
<p>ตรวจสอบ volumes:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker volume <span class="token function">ls</span>
</code></pre>
<hr>
<h2 id="ขั้นตอนที่-4-การตั้งค่าบัญชีผู้ดูแลระบบ">ขั้นตอนที่ 4: การตั้งค่าบัญชีผู้ดูแลระบบ</h2>
<h3 id="การดึงรหัสผ่านเริ่มต้น">4.1 การดึงรหัสผ่านเริ่มต้น</h3>
<p>GitLab จะสร้างรหัสผ่านชั่วคราวสำหรับบัญชี root โดยอัตโนมัติในครั้งแรกที่เริ่มระบบ</p>
<p><strong>วิธีดูรหัสผ่าน:</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab <span class="token function">grep</span> <span class="token string">'Password:'</span> /etc/gitlab/initial_root_password
</code></pre>
<p><strong>คำอธิบายคำสั่ง:</strong></p>
<ul>
<li><code>docker exec</code>: รันคำสั่งภายใน container ที่กำลังทำงาน</li>
<li><code>gitlab</code>: ชื่อของ container</li>
<li><code>grep 'Password:'</code>: ค้นหาบรรทัดที่มีคำว่า “Password:”</li>
<li><code>/etc/gitlab/initial_root_password</code>: ไฟล์ที่เก็บรหัสผ่านชั่วคราว</li>
</ul>
<p><strong>ผลลัพธ์:</strong></p>
<pre><code>Password: 5iveL!fe=AweSome+GitLab123
</code></pre>
<p><strong>หมายเหตุสำคัญ:</strong></p>
<ul>
<li>รหัสผ่านนี้จะถูกสร้างขึ้นเพียงครั้งเดียวเมื่อ GitLab เริ่มครั้งแรก</li>
<li>ไฟล์นี้จะถูกลบอัตโนมัติหลังจาก 24 ชั่วโมง เพื่อความปลอดภัย</li>
<li>ควรบันทึกรหัสผ่านนี้ไว้ในที่ปลอดภัยก่อนเข้าสู่ระบบ</li>
</ul>
<p><strong>กรณีที่ไฟล์ถูกลบไปแล้ว:</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Reset password ด้วยคำสั่ง GitLab Rails Console</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> -it gitlab gitlab-rake <span class="token string">"gitlab:password:reset[root]"</span>
</code></pre>
<h3 id="การเข้าสู่ระบบครั้งแรก">4.2 การเข้าสู่ระบบครั้งแรก</h3>
<h4 id="ขั้นตอนที่-1-เข้าสู่หน้า-login">ขั้นตอนที่ 1: เข้าสู่หน้า Login</h4>
<ol>
<li>เปิดเว็บเบราว์เซอร์ (แนะนำ Chrome, Firefox, หรือ Edge)</li>
<li>เข้าสู่ URL ที่ตั้งค่าไว้:
<ul>
<li>ถ้าใช้ IP: <code>http://192.168.1.100</code></li>
<li>ถ้าใช้ Domain: <code>http://gitlab.example.com</code></li>
</ul>
</li>
<li>หน้า GitLab Sign In จะปรากฏขึ้น</li>
</ol>
<h4 id="ขั้นตอนที่-2-กรอกข้อมูลเข้าสู่ระบบ">ขั้นตอนที่ 2: กรอกข้อมูลเข้าสู่ระบบ</h4>
<pre><code>Username: root
Password: [รหัสผ่านจากขั้นตอน 4.1]
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li>บัญชี <code>root</code> คือบัญชีผู้ดูแลระบบสูงสุด (Administrator)</li>
<li>มีสิทธิ์ในการจัดการทุกอย่างในระบบ</li>
</ul>
<h4 id="ขั้นตอนที่-3-ตั้งรหัสผ่านใหม่">ขั้นตอนที่ 3: ตั้งรหัสผ่านใหม่</h4>
<p>ระบบจะนำคุณไปยังหน้า “Change your password” ทันที</p>
<p><strong>แนวทางการตั้งรหัสผ่านที่ปลอดภัย:</strong></p>
<ul>
<li>ความยาวอย่างน้อย 12 ตัวอักษร</li>
<li>ประกอบด้วยตัวพิมพ์ใหญ่, ตัวพิมพ์เล็ก, ตัวเลข, และอักขระพิเศษ</li>
<li>ไม่ใช้คำที่อยู่ในพจนานุกรม</li>
<li>ไม่ใช้ข้อมูลส่วนตัวที่เดาได้ง่าย</li>
</ul>
<p><strong>ตัวอย่างรหัสผ่านที่แข็งแรง:</strong></p>
<pre><code>Gitlab@2024!Secure#Pass
MyC0mpany$GitLab%2024
</code></pre>
<h4 id="ขั้นตอนที่-4-ยืนยันการเปลี่ยนแปลง">ขั้นตอนที่ 4: ยืนยันการเปลี่ยนแปลง</h4>
<ol>
<li>กรอกรหัสผ่านใหม่ในช่อง “New password”</li>
<li>กรอกรหัสผ่านซ้ำในช่อง “Confirm new password”</li>
<li>คลิกปุ่ม “Change your password”</li>
</ol>
<h3 id="การตั้งค่าเพิ่มเติมหลังเข้าสู่ระบบ">4.3 การตั้งค่าเพิ่มเติมหลังเข้าสู่ระบบ</h3>
<p>หลังจากเข้าสู่ระบบสำเร็จ ควรทำการตั้งค่าต่อไปนี้:</p>
<h4 id="ตั้งค่าข้อมูลส่วนตัว">ตั้งค่าข้อมูลส่วนตัว</h4>
<ol>
<li>คลิกที่ Avatar มุมขวาบน &gt; Settings</li>
<li>กรอกข้อมูล:
<ul>
<li>Full name</li>
<li>Email address</li>
<li>Public email (ใช้แสดงใน commits)</li>
<li>Time zone</li>
</ul>
</li>
</ol>
<h4 id="เปิดใช้งาน-two-factor-authentication-แนะนำ">เปิดใช้งาน Two-Factor Authentication (แนะนำ)</h4>
<ol>
<li>Settings &gt; Account &gt; Two-Factor Authentication</li>
<li>สแกน QR code ด้วยแอพ Authenticator (Google Authenticator, Authy)</li>
<li>กรอกรหัสยืนยัน</li>
<li>บันทึก Recovery codes ไว้ในที่ปลอดภัย</li>
</ol>
<hr>
<h2 id="ขั้นตอนที่-5-การลงทะเบียน-gitlab-runner">ขั้นตอนที่ 5: การลงทะเบียน GitLab Runner</h2>
<p>GitLab Runner คือส่วนที่จะรับผิดชอบในการรัน CI/CD jobs การลงทะเบียนคือการเชื่อมโยง Runner เข้ากับ GitLab Server</p>
<h3 id="ความเข้าใจเกี่ยวกับ-runner-types">5.1 ความเข้าใจเกี่ยวกับ Runner Types</h3>
<p>GitLab รองรับ Runner 3 ประเภท:</p>
<p><strong>1. Shared Runners</strong></p>
<ul>
<li>ใช้งานได้กับทุก projects ในระบบ</li>
<li>เหมาะสำหรับงานทั่วไป</li>
<li>ตั้งค่าโดย Administrator</li>
</ul>
<p><strong>2. Group Runners</strong></p>
<ul>
<li>ใช้งานได้กับทุก projects ใน group</li>
<li>เหมาะสำหรับทีมหรือแผนกเดียวกัน</li>
</ul>
<p><strong>3. Project Runners</strong></p>
<ul>
<li>ใช้งานได้เฉพาะ project ที่กำหนด</li>
<li>เหมาะสำหรับงานที่ต้องการ environment พิเศษ</li>
</ul>
<h3 id="การขอ-token-สำหรับลงทะเบียน">5.2 การขอ Token สำหรับลงทะเบียน</h3>
<h4 id="สำหรับ-shared-runner-instance-wide">สำหรับ Shared Runner (Instance-wide)</h4>
<p><strong>ขั้นตอน:</strong></p>
<ol>
<li>เข้าสู่ระบบ GitLab ด้วยบัญชี root</li>
<li>คลิกที่ไอคอนเมนู (☰) มุมซ้ายบน</li>
<li>เลือก <strong>Admin Area</strong></li>
<li>ในเมนูด้านซ้าย: <strong>CI/CD</strong> &gt; <strong>Runners</strong></li>
<li>ในส่วน “Register an instance runner” คุณจะพบ:
<ul>
<li><strong>Registration URL</strong>: <code>http://gitlab</code> (สำหรับ internal communication)</li>
<li><strong>Registration Token</strong>: รหัสที่ใช้ในการลงทะเบียน (เช่น <code>GR1348941xxxxxxxxxx</code>)</li>
</ul>
</li>
</ol>
<p><strong>บันทึกข้อมูลทั้งสองนี้ไว้</strong></p>
<h4 id="สำหรับ-project-runner">สำหรับ Project Runner</h4>
<p><strong>ขั้นตอน:</strong></p>
<ol>
<li>เข้าสู่ Project ที่ต้องการ</li>
<li>ไปที่ <strong>Settings</strong> &gt; <strong>CI/CD</strong></li>
<li>ขยายส่วน <strong>Runners</strong></li>
<li>ในส่วน “Project runners” จะมี URL และ Token</li>
</ol>
<h3 id="การลงทะเบียน-runner-ผ่าน-container">5.3 การลงทะเบียน Runner ผ่าน Container</h3>
<p>เริ่มกระบวนการลงทะเบียน:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">exec</span> -it gitlab-runner gitlab-runner register
</code></pre>
<p><strong>คำอธิบายคำสั่ง:</strong></p>
<ul>
<li><code>-it</code>: Interactive mode (สามารถโต้ตอบได้)</li>
<li><code>gitlab-runner register</code>: คำสั่งย่อยสำหรับลงทะเบียน</li>
</ul>
<h3 id="การตอบคำถามในขั้นตอนลงทะเบียน">5.4 การตอบคำถามในขั้นตอนลงทะเบียน</h3>
<p>ระบบจะถามคำถามตามลำดับ ให้ตอบดังนี้:</p>
<h4 id="คำถามที่-1-gitlab-instance-url">คำถามที่ 1: GitLab Instance URL</h4>
<pre><code>Enter the GitLab instance URL (for example, https://gitlab.com/):
http://gitlab
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li>ใช้ชื่อ service <code>gitlab</code> ตามที่กำหนดใน docker-compose.yml</li>
<li>Docker จะ resolve ชื่อนี้ผ่าน internal DNS</li>
<li><strong>ไม่ต้อง</strong> ใช้ external URL หรือ IP Address</li>
</ul>
<p><strong>ทำไมไม่ใช้ external URL?</strong></p>
<ul>
<li>Containers อยู่ใน network เดียวกัน สื่อสารกันได้โดยตรง</li>
<li>รวดเร็วกว่า (ไม่ต้องผ่าน reverse proxy หรือ firewall)</li>
<li>ปลอดภัยกว่า (traffic ไม่ออกนอก host)</li>
</ul>
<h4 id="คำถามที่-2-registration-token">คำถามที่ 2: Registration Token</h4>
<pre><code>Enter the registration token:
GR1348941xxxxxxxxxx
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li>คัดลอก Token จากหน้า Admin Area &gt; CI/CD &gt; Runners</li>
<li>Token นี้ใช้เพื่อยืนยันตัวตนของ Runner</li>
<li>แต่ละ Runner type จะมี Token ต่างกัน</li>
</ul>
<p><strong>หมายเหตุด้านความปลอดภัย:</strong></p>
<ul>
<li>เก็บ Token เป็นความลับ</li>
<li>อย่าแชร์ในที่สาธารณะ</li>
<li>สามารถ revoke และสร้างใหม่ได้ถ้าจำเป็น</li>
</ul>
<h4 id="คำถามที่-3-runner-description">คำถามที่ 3: Runner Description</h4>
<pre><code>Enter a description for the runner:
docker-executor
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li>ชื่อที่ใช้อธิบาย Runner นี้</li>
<li>จะแสดงในหน้า Runners list</li>
<li>ควรตั้งชื่อให้สื่อความหมาย</li>
</ul>
<p><strong>ตัวอย่างชื่อที่ดี:</strong></p>
<ul>
<li><code>production-docker-runner</code></li>
<li><code>staging-linux-executor</code></li>
<li><code>docker-build-runner-01</code></li>
<li><code>shared-runner-ubuntu-22</code></li>
</ul>
<h4 id="คำถามที่-4-runner-tags">คำถามที่ 4: Runner Tags</h4>
<pre><code>Enter tags for the runner (comma-separated):
docker,linux,ci
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li>Tags ใช้สำหรับกำหนดว่า job ใดจะรันบน runner ใด</li>
<li>สามารถกำหนดหลาย tags โดยใช้ comma คั่น</li>
<li>ใน <code>.gitlab-ci.yml</code> สามารถระบุ tags เพื่อเลือก runner</li>
</ul>
<p><strong>ตัวอย่างการใช้งาน tags ในไฟล์ <code>.gitlab-ci.yml</code>:</strong></p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">build_job</span><span class="token punctuation">:</span>
  <span class="token key atrule">stage</span><span class="token punctuation">:</span> build
  <span class="token key atrule">tags</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> docker
    <span class="token punctuation">-</span> linux
  <span class="token key atrule">script</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> docker build <span class="token punctuation">-</span>t myapp .
</code></pre>
<p><strong>แนวทางการตั้ง tags:</strong></p>
<ul>
<li><code>docker</code>: ระบุว่าใช้ Docker executor</li>
<li><code>linux</code>: ระบุว่ารันบน Linux OS</li>
<li><code>ci</code>: ระบุว่าใช้สำหรับ CI pipeline</li>
<li>เพิ่ม tags ตามความเหมาะสม เช่น <code>production</code>, <code>staging</code>, <code>arm64</code>, <code>gpu</code></li>
</ul>
<h4 id="คำถามที่-5-runner-executor">คำถามที่ 5: Runner Executor</h4>
<pre><code>Enter an executor: custom, docker, parallels, shell, ssh, docker-autoscaler, 
virtualbox, docker+machine, instance, kubernetes:
docker
</code></pre>
<p><strong>คำอธิบาย executors ที่มี:</strong></p>
<p><strong>1. docker</strong> (แนะนำ)</p>
<ul>
<li>รัน jobs ภายใน Docker containers</li>
<li>แต่ละ job จะได้ environment ที่สะอาด (clean)</li>
<li>รองรับ caching และ artifacts</li>
<li>เหมาะสำหรับ CI/CD ทั่วไป</li>
</ul>
<p><strong>2. shell</strong></p>
<ul>
<li>รัน commands โดยตรงบน host</li>
<li>เร็วที่สุด แต่ไม่มี isolation</li>
<li>เสี่ยงต่อการ contamination ระหว่าง jobs</li>
</ul>
<p><strong>3. kubernetes</strong></p>
<ul>
<li>รัน jobs ใน Kubernetes pods</li>
<li>เหมาะสำหรับ cloud-native environments</li>
<li>มี auto-scaling</li>
</ul>
<p><strong>4. ssh</strong></p>
<ul>
<li>รัน jobs บน remote machine ผ่าน SSH</li>
<li>เหมาะสำหรับ testing บน specific hardware</li>
</ul>
<p><strong>5. virtualbox</strong></p>
<ul>
<li>รัน jobs ใน virtual machines</li>
<li>isolation สูงแต่ใช้ resources มาก</li>
</ul>
<p><strong>เลือก <code>docker</code> เพราะ:</strong></p>
<ul>
<li>มี isolation ที่ดี</li>
<li>ใช้ resources น้อยกว่า VM</li>
<li>สามารถระบุ Docker image ที่ต้องการได้</li>
<li>รองรับ Docker-in-Docker สำหรับ build images</li>
</ul>
<h4 id="คำถามที่-6-default-docker-image">คำถามที่ 6: Default Docker Image</h4>
<pre><code>Enter the default Docker image (for example, ruby:2.7):
alpine:latest
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li>Image นี้จะถูกใช้เมื่อ job ไม่ระบุ image</li>
<li>ควรเลือก image ที่เล็กและมีเครื่องมือพื้นฐาน</li>
</ul>
<p><strong>ตัวเลือก Docker Images ที่นิยม:</strong></p>
<pre><code>alpine:latest           # เล็กที่สุด (~5MB), เหมาะสำหรับ basic tasks
ubuntu:22.04           # ใช้งานง่าย, มีเครื่องมือครบ
node:18-alpine         # สำหรับ Node.js projects
python:3.11-slim       # สำหรับ Python projects
maven:3.9-eclipse-temurin  # สำหรับ Java/Maven projects
golang:1.21-alpine     # สำหรับ Go projects
</code></pre>
<p><strong>การ override default image ใน <code>.gitlab-ci.yml</code>:</strong></p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">build_job</span><span class="token punctuation">:</span>
  <span class="token key atrule">image</span><span class="token punctuation">:</span> node<span class="token punctuation">:</span>18<span class="token punctuation">-</span>alpine
  <span class="token key atrule">script</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> npm install
    <span class="token punctuation">-</span> npm run build
</code></pre>
<h3 id="ผลลัพธ์การลงทะเบียน">5.5 ผลลัพธ์การลงทะเบียน</h3>
<p>หลังจากตอบคำถามครบ คุณจะเห็นข้อความยืนยัน:</p>
<pre><code>Runner registered successfully. Feel free to start it, but if it's running 
already the config should be automatically reloaded!
</code></pre>
<p><strong>ความหมาย:</strong></p>
<ul>
<li>Runner ถูกลงทะเบียนสำเร็จ</li>
<li>ข้อมูลถูกบันทึกในไฟล์ <code>/etc/gitlab-runner/config.toml</code></li>
<li>Runner จะเริ่มทำงานอัตโนมัติ</li>
</ul>
<h3 id="การตรวจสอบการลงทะเบียน">5.6 การตรวจสอบการลงทะเบียน</h3>
<h4 id="ตรวจสอบจากหน้า-gitlab-web-interface">ตรวจสอบจากหน้า GitLab Web Interface</h4>
<ol>
<li>กลับไปที่ Admin Area &gt; CI/CD &gt; Runners</li>
<li>คุณจะเห็น Runner ที่เพิ่งลงทะเบียนในรายการ</li>
<li>สถานะควรเป็น:
<ul>
<li>🟢 <strong>Online</strong>: Runner พร้อมรับงาน</li>
<li>🔴 <strong>Offline</strong>: Runner ไม่สามารถติดต่อ GitLab ได้</li>
</ul>
</li>
</ol>
<h4 id="ตรวจสอบจาก-command-line">ตรวจสอบจาก Command Line</h4>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># ดูรายการ Runners ทั้งหมด</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner gitlab-runner list

<span class="token comment"># ตรวจสอบสถานะ</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner gitlab-runner status

<span class="token comment"># ดูไฟล์ config</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner <span class="token function">cat</span> /etc/gitlab-runner/config.toml
</code></pre>
<p><strong>ตัวอย่างไฟล์ config.toml:</strong></p>
<pre class=" language-toml"><code class="prism  language-toml">concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-executor"
  url = "http://gitlab"
  token = "xxxxxxxxxxxxxxxxxxxx"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
</code></pre>
<p><strong>คำอธิบายการตั้งค่าสำคัญ:</strong></p>
<ul>
<li><code>concurrent = 1</code>: จำนวน jobs สูงสุดที่รันพร้อมกันได้</li>
<li><code>check_interval = 0</code>: ความถี่ในการตรวจสอบ jobs ใหม่</li>
<li><code>privileged = false</code>: ไม่ให้ container มีสิทธิ์สูง (recommended)</li>
<li><code>volumes = ["/cache"]</code>: mount cache volume</li>
</ul>
<h3 id="การปรับแต่งการตั้งค่า-runner-ขั้นสูง">5.7 การปรับแต่งการตั้งค่า Runner (ขั้นสูง)</h3>
<h4 id="เพิ่มจำนวน-concurrent-jobs">เพิ่มจำนวน concurrent jobs</h4>
<p>แก้ไฟล์ config.toml:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">exec</span> -it gitlab-runner <span class="token function">nano</span> /etc/gitlab-runner/config.toml
</code></pre>
<p>เปลี่ยนค่า:</p>
<pre class=" language-toml"><code class="prism  language-toml">concurrent = 4  # รัน 4 jobs พร้อมกัน
</code></pre>
<h4 id="เพิ่ม-docker-volumes">เพิ่ม Docker volumes</h4>
<pre class=" language-toml"><code class="prism  language-toml">[runners.docker]
  volumes = [
    "/cache",
    "/var/run/docker.sock:/var/run/docker.sock",
    "/builds:/builds"
  ]
</code></pre>
<h4 id="ตั้งค่า-docker-pull-policy">ตั้งค่า Docker pull policy</h4>
<pre class=" language-toml"><code class="prism  language-toml">[runners.docker]
  pull_policy = ["if-not-present", "always"]
</code></pre>
<p><strong>Pull policies:</strong></p>
<ul>
<li><code>always</code>: ดึง image ใหม่ทุกครั้ง</li>
<li><code>if-not-present</code>: ใช้ local image ถ้ามี</li>
<li><code>never</code>: ไม่ดึง image (ต้องมีอยู่แล้ว)</li>
</ul>
<h4 id="กำหนด-resource-limits">กำหนด resource limits</h4>
<pre class=" language-toml"><code class="prism  language-toml">[runners.docker]
  cpus = "2"
  memory = "4g"
  memory_swap = "4g"
  memory_reservation = "2g"
</code></pre>
<h4 id="หลังจากแก้ไข-config-ให้-restart-runner">หลังจากแก้ไข config ให้ restart runner</h4>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker restart gitlab-runner
</code></pre>
<hr>
<h2 id="ขั้นตอนที่-6-การทดสอบ-cicd-pipeline">ขั้นตอนที่ 6: การทดสอบ CI/CD Pipeline</h2>
<h3 id="สร้าง-test-project">6.1 สร้าง Test Project</h3>
<ol>
<li>เข้าสู่ GitLab</li>
<li>คลิก “New project”</li>
<li>เลือก “Create blank project”</li>
<li>กรอกข้อมูล:
<ul>
<li>Project name: <code>test-ci-cd</code></li>
<li>Visibility Level: Private</li>
<li>Initialize with README: ✓ เลือก</li>
</ul>
</li>
</ol>
<h3 id="สร้างไฟล์-.gitlab-ci.yml">6.2 สร้างไฟล์ .gitlab-ci.yml</h3>
<p>สร้างไฟล์ <code>.gitlab-ci.yml</code> ใน root directory ของ project:</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token comment"># กำหนดขั้นตอน (stages) ของ pipeline</span>
<span class="token key atrule">stages</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> build
  <span class="token punctuation">-</span> test
  <span class="token punctuation">-</span> deploy

<span class="token comment"># Job 1: Build stage</span>
<span class="token key atrule">build_job</span><span class="token punctuation">:</span>
  <span class="token key atrule">stage</span><span class="token punctuation">:</span> build
  <span class="token comment"># กำหนด Docker image ที่จะใช้รัน job นี้</span>
  <span class="token key atrule">image</span><span class="token punctuation">:</span> alpine<span class="token punctuation">:</span>latest
  <span class="token comment"># กำหนด tags เพื่อเลือก runner</span>
  <span class="token key atrule">tags</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> docker
  <span class="token comment"># คำสั่งที่จะรัน</span>
  <span class="token key atrule">script</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> echo "Building the application<span class="token punctuation">...</span>"
    <span class="token punctuation">-</span> echo "Build version 1.0.0"
    <span class="token punctuation">-</span> mkdir <span class="token punctuation">-</span>p build/
    <span class="token punctuation">-</span> echo "Build completed" <span class="token punctuation">&gt;</span> build/output.txt
  <span class="token comment"># เก็บไฟล์ที่สร้างไว้ใช้ในขั้นตอนถัดไป</span>
  <span class="token key atrule">artifacts</span><span class="token punctuation">:</span>
    <span class="token key atrule">paths</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> build/
    <span class="token key atrule">expire_in</span><span class="token punctuation">:</span> 1 hour

<span class="token comment"># Job 2: Test stage</span>
<span class="token key atrule">test_job</span><span class="token punctuation">:</span>
  <span class="token key atrule">stage</span><span class="token punctuation">:</span> test
  <span class="token key atrule">image</span><span class="token punctuation">:</span> alpine<span class="token punctuation">:</span>latest
  <span class="token key atrule">tags</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> docker
  <span class="token key atrule">script</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> echo "Running tests<span class="token punctuation">...</span>"
    <span class="token punctuation">-</span> cat build/output.txt
    <span class="token punctuation">-</span> echo "All tests passed<span class="token tag">!"</span>
  <span class="token comment"># job นี้ขึ้นอยู่กับ artifacts จาก build_job</span>
  <span class="token key atrule">dependencies</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> build_job

<span class="token comment"># Job 3: Deploy stage (รันเฉพาะ main branch)</span>
<span class="token key atrule">deploy_job</span><span class="token punctuation">:</span>
  <span class="token key atrule">stage</span><span class="token punctuation">:</span> deploy
  <span class="token key atrule">image</span><span class="token punctuation">:</span> alpine<span class="token punctuation">:</span>latest
  <span class="token key atrule">tags</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> docker
  <span class="token key atrule">script</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> echo "Deploying application<span class="token punctuation">...</span>"
    <span class="token punctuation">-</span> echo "Deployment successful<span class="token tag">!"</span>
  <span class="token comment"># กำหนดเงื่อนไขว่าจะรันเมื่อไหร่</span>
  <span class="token key atrule">only</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> main
  <span class="token comment"># ต้องยืนยันด้วยตนเองก่อนจะรัน</span>
  <span class="token key atrule">when</span><span class="token punctuation">:</span> manual
</code></pre>
<h3 id="commit-และ-push">6.3 Commit และ Push</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">git</span> add .gitlab-ci.yml
<span class="token function">git</span> commit -m <span class="token string">"Add CI/CD pipeline"</span>
<span class="token function">git</span> push origin main
</code></pre>
<h3 id="ตรวจสอบ-pipeline">6.4 ตรวจสอบ Pipeline</h3>
<ol>
<li>ไปที่ Project &gt; CI/CD &gt; Pipelines</li>
<li>คุณจะเห็น pipeline ใหม่กำลังทำงาน</li>
<li>คลิกเข้าไปดูรายละเอียดแต่ละ job</li>
<li>ตรวจสอบ logs ของแต่ละ job</li>
</ol>
<hr>
<h2 id="การบำรุงรักษาและจัดการ">การบำรุงรักษาและจัดการ</h2>
<h3 id="การ-backup-ข้อมูล">การ Backup ข้อมูล</h3>
<h4 id="backup-gitlab-data">Backup GitLab Data</h4>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Backup ทั้งหมด</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-backup create

<span class="token comment"># Backup จะถูกเก็บที่</span>
./gitlab/data/backups/
</code></pre>
<h4 id="backup-configuration">Backup Configuration</h4>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Backup ไฟล์ config</span>
<span class="token function">sudo</span> <span class="token function">tar</span> -czf gitlab-config-backup.tar.gz ./gitlab/config/
</code></pre>
<h3 id="การ-restore">การ Restore</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># หยุด services ที่เกี่ยวข้อง</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-ctl stop puma
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-ctl stop sidekiq

<span class="token comment"># Restore backup</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-backup restore BACKUP<span class="token operator">=</span><span class="token operator">&lt;</span>timestamp<span class="token operator">&gt;</span>

<span class="token comment"># Restart services</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-ctl restart
</code></pre>
<h3 id="การอัพเกรด-gitlab">การอัพเกรด GitLab</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Pull image version ใหม่</span>
<span class="token function">sudo</span> docker compose pull gitlab

<span class="token comment"># Restart เพื่ออัพเกรด</span>
<span class="token function">sudo</span> docker compose up -d gitlab

<span class="token comment"># ตรวจสอบ logs</span>
<span class="token function">sudo</span> docker logs -f gitlab
</code></pre>
<h3 id="การตรวจสอบ-health">การตรวจสอบ Health</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># ตรวจสอบสถานะ services ทั้งหมด</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-ctl status

<span class="token comment"># ตรวจสอบ logs</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-ctl <span class="token function">tail</span>

<span class="token comment"># ดู disk usage</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab <span class="token function">du</span> -sh /var/opt/gitlab/*
</code></pre>
<h3 id="การจัดการ-runner">การจัดการ Runner</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Unregister runner</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner gitlab-runner unregister --name <span class="token string">"docker-executor"</span>

<span class="token comment"># Verify runner</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner gitlab-runner verify

<span class="token comment"># Restart runner</span>
<span class="token function">sudo</span> docker restart gitlab-runner
</code></pre>
<hr>
<h2 id="การแก้ไขปัญหาเบื้องต้น">การแก้ไขปัญหาเบื้องต้น</h2>
<h3 id="ปัญหา-gitlab-ไม่สามารถเข้าถึงได้">ปัญหา: GitLab ไม่สามารถเข้าถึงได้</h3>
<p><strong>วิธีแก้:</strong></p>
<ol>
<li>ตรวจสอบว่า container ทำงาน:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">ps</span> <span class="token operator">|</span> <span class="token function">grep</span> gitlab
</code></pre>
<ol start="2">
<li>ตรวจสอบ logs:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker logs gitlab <span class="token operator">|</span> <span class="token function">tail</span> -100
</code></pre>
<ol start="3">
<li>ตรวจสอบ ports:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">netstat</span> -tulpn <span class="token operator">|</span> <span class="token function">grep</span> -E <span class="token string">'(80|443|22)'</span>
</code></pre>
<ol start="4">
<li>ตรวจสอบ firewall:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> ufw status
<span class="token function">sudo</span> ufw allow 80/tcp
<span class="token function">sudo</span> ufw allow 443/tcp
<span class="token function">sudo</span> ufw allow 22/tcp
</code></pre>
<h3 id="ปัญหา-runner-ไม่สามารถลงทะเบียนได้">ปัญหา: Runner ไม่สามารถลงทะเบียนได้</h3>
<p><strong>วิธีแก้:</strong></p>
<ol>
<li>ตรวจสอบว่า Runner container ทำงาน:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">ps</span> <span class="token operator">|</span> <span class="token function">grep</span> runner
</code></pre>
<ol start="2">
<li>ตรวจสอบการเชื่อมต่อระหว่าง containers:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner <span class="token function">ping</span> -c 4 gitlab
</code></pre>
<ol start="3">
<li>
<p>ตรวจสอบ Token:</p>
<ul>
<li>Token อาจหมดอายุหรือถูกเปลี่ยน</li>
<li>ขอ Token ใหม่จาก Admin Area</li>
</ul>
</li>
<li>
<p>ตรวจสอบ network:</p>
</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker network inspect gitlab-ci-cd_default
</code></pre>
<h3 id="ปัญหา-pipeline-ล้มเหลว">ปัญหา: Pipeline ล้มเหลว</h3>
<p><strong>วิธีแก้:</strong></p>
<ol>
<li>
<p>ดู job logs ละเอียด:</p>
<ul>
<li>คลิกเข้าไปที่ failed job</li>
<li>อ่าน error messages</li>
</ul>
</li>
<li>
<p>ตรวจสอบ Runner status:</p>
<ul>
<li>Admin Area &gt; CI/CD &gt; Runners</li>
<li>ตรวจสอบว่า Runner online</li>
</ul>
</li>
<li>
<p>ตรวจสอบ tags:</p>
<ul>
<li>ใน <code>.gitlab-ci.yml</code> ตรวจสอบว่า tags ตรงกับ Runner</li>
</ul>
</li>
<li>
<p>ทดสอบ locally:</p>
</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># ติดตั้ง gitlab-runner locally</span>
gitlab-runner <span class="token function">exec</span> docker build_job
</code></pre>
<h3 id="ปัญหา-docker-socket-permission-denied">ปัญหา: Docker Socket Permission Denied</h3>
<p><strong>วิธีแก้:</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># เพิ่ม privileged mode ใน docker-compose.yml</span>
gitlab-runner:
  privileged: <span class="token boolean">true</span>
  
<span class="token comment"># หรือเพิ่ม user ไปยัง docker group</span>
<span class="token function">sudo</span> <span class="token function">usermod</span> -aG docker gitlab-runner
</code></pre>
<h3 id="ปัญหา-out-of-disk-space">ปัญหา: Out of Disk Space</h3>
<p><strong>วิธีแก้:</strong></p>
<ol>
<li>ลบ Docker objects ที่ไม่ใช้:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker system prune -a --volumes
</code></pre>
<ol start="2">
<li>ลบ old artifacts:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># ใน GitLab: Admin Area &gt; Settings &gt; CI/CD</span>
<span class="token comment"># ตั้งค่า "Default artifacts expiration"</span>
</code></pre>
<ol start="3">
<li>ลบ old backups:</li>
</ol>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># เก็บแค่ 7 วันล่าสุด</span>
<span class="token function">find</span> ./gitlab/data/backups/ -type f -mtime +7 -delete
</code></pre>
<hr>
<h2 id="ภาคผนวก-คำสั่งที่เป็นประโยชน์">ภาคผนวก: คำสั่งที่เป็นประโยชน์</h2>
<h3 id="คำสั่ง-docker-compose">คำสั่ง Docker Compose</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># เริ่มต้นระบบ</span>
<span class="token function">sudo</span> docker compose up -d

<span class="token comment"># หยุดระบบ</span>
<span class="token function">sudo</span> docker compose down

<span class="token comment"># Restart services</span>
<span class="token function">sudo</span> docker compose restart

<span class="token comment"># ดู logs</span>
<span class="token function">sudo</span> docker compose logs -f

<span class="token comment"># Pull images ใหม่</span>
<span class="token function">sudo</span> docker compose pull

<span class="token comment"># Build และ restart</span>
<span class="token function">sudo</span> docker compose up -d --build
</code></pre>
<h3 id="คำสั่ง-gitlab">คำสั่ง GitLab</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Reconfigure GitLab</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-ctl reconfigure

<span class="token comment"># Restart all services</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-ctl restart

<span class="token comment"># Check configuration</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-rake gitlab:check

<span class="token comment"># Console (Ruby Rails)</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> -it gitlab gitlab-rails console

<span class="token comment"># Reset root password</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab gitlab-rake <span class="token string">"gitlab:password:reset[root]"</span>
</code></pre>
<h3 id="คำสั่ง-gitlab-runner">คำสั่ง GitLab Runner</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># List runners</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner gitlab-runner list

<span class="token comment"># Verify runner</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner gitlab-runner verify

<span class="token comment"># Run runner manually</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner gitlab-runner run

<span class="token comment"># Update runner</span>
<span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab-runner gitlab-runner stop
<span class="token function">sudo</span> docker compose pull gitlab-runner
<span class="token function">sudo</span> docker compose up -d gitlab-runner
</code></pre>
<hr>
<h2 id="สรุป">สรุป</h2>
<p>เอกสารนี้ได้อธิบายขั้นตอนการติดตั้งและกำหนดค่า GitLab CE พร้อม GitLab Runner บน Docker อย่างครบถ้วน รวมถึง:</p>
<p>✅ การเตรียมระบบและโครงสร้างไดเรกทอรี<br>
✅ การสร้างไฟล์ Docker Compose พร้อมคำอธิบายละเอียด<br>
✅ การตั้งค่าบัญชีผู้ดูแลระบบ<br>
✅ การลงทะเบียนและกำหนดค่า Runner<br>
✅ การทดสอบ CI/CD Pipeline<br>
✅ การบำรุงรักษาและแก้ไขปัญหา</p>
<p>ระบบนี้พร้อมสำหรับการใช้งาน CI/CD ในสภาพแวดล้อม Production หรือ Development</p>

