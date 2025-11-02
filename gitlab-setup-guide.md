---


---

<h1 id="คู่มือการติดตั้ง-gitlab-ce-และ-gitlab-runner-บน-docker">คู่มือการติดตั้ง GitLab CE และ GitLab Runner บน Docker</h1>
<h2 id="บทนำ">บทนำ</h2>
<p>เอกสารนี้จะอธิบายขั้นตอนการติดตั้ง GitLab Community Edition (CE) พร้อมกับ GitLab Runner โดยใช้ Docker Compose สำหรับการทำ CI/CD (Continuous Integration/Continuous Deployment) บนระบบปฏิบัติการ Ubuntu Server</p>
<hr>
<h2 id="ข้อกำหนดเบื้องต้น">ข้อกำหนดเบื้องต้น</h2>
<p>ก่อนเริ่มการติดตั้ง กรุณาตรวจสอบให้แน่ใจว่าระบบของคุณมีส่วนประกอบต่อไปนี้:</p>
<ul>
<li>Docker Engine เวอร์ชันล่าสุด</li>
<li>Docker Compose เวอร์ชันล่าสุด</li>
<li>Ubuntu Server (แนะนำ 20.04 LTS ขึ้นไป)</li>
<li>สิทธิ์ผู้ดูแลระบบ (sudo)</li>
</ul>
<hr>
<h2 id="ขั้นตอนที่-1-การเตรียมโครงสร้างไดเรกทอรี">ขั้นตอนที่ 1: การเตรียมโครงสร้างไดเรกทอรี</h2>
<h3 id="สร้างไดเรกทอรีโครงการ">1.1 สร้างไดเรกทอรีโครงการ</h3>
<p>สร้างไดเรกทอรีหลักสำหรับโครงการ:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">mkdir</span> gitlab-ci-cd
<span class="token function">cd</span> gitlab-ci-cd
</code></pre>
<h3 id="สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล">1.2 สร้างไดเรกทอรีสำหรับจัดเก็บข้อมูล</h3>
<p>สร้างไดเรกทอรีย่อยสำหรับจัดเก็บข้อมูลของ GitLab และ GitLab Runner:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">mkdir</span> -p gitlab/config gitlab/logs gitlab/data
<span class="token function">mkdir</span> -p gitlab-runner/config
</code></pre>
<p><strong>คำอธิบาย:</strong></p>
<ul>
<li><code>gitlab/config</code> - เก็บไฟล์การตั้งค่า</li>
<li><code>gitlab/logs</code> - เก็บไฟล์บันทึกการทำงาน</li>
<li><code>gitlab/data</code> - เก็บข้อมูลหลักของ GitLab</li>
<li><code>gitlab-runner/config</code> - เก็บการตั้งค่าของ Runner</li>
</ul>
<hr>
<h2 id="ขั้นตอนที่-2-การสร้างไฟล์-docker-compose">ขั้นตอนที่ 2: การสร้างไฟล์ Docker Compose</h2>
<p>สร้างไฟล์ <code>docker-compose.yml</code> ในไดเรกทอรี <code>gitlab-ci-cd</code>:</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">version</span><span class="token punctuation">:</span> <span class="token string">"3.7"</span>
<span class="token key atrule">services</span><span class="token punctuation">:</span>
  <span class="token key atrule">gitlab</span><span class="token punctuation">:</span>
    <span class="token key atrule">image</span><span class="token punctuation">:</span> <span class="token string">'gitlab/gitlab-ce:latest'</span>
    <span class="token key atrule">container_name</span><span class="token punctuation">:</span> gitlab
    <span class="token key atrule">restart</span><span class="token punctuation">:</span> always
    <span class="token key atrule">hostname</span><span class="token punctuation">:</span> <span class="token string">'gitlab.example.com'</span>
    <span class="token key atrule">environment</span><span class="token punctuation">:</span>
      <span class="token key atrule">GITLAB_OMNIBUS_CONFIG</span><span class="token punctuation">:</span> <span class="token punctuation">|</span><span class="token scalar string">
        external_url 'http://gitlab.example.com'</span>
    <span class="token key atrule">ports</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> <span class="token string">'80:80'</span>
      <span class="token punctuation">-</span> <span class="token string">'443:443'</span>
      <span class="token punctuation">-</span> <span class="token string">'22:22'</span>
    <span class="token key atrule">volumes</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> <span class="token string">'./gitlab/config:/etc/gitlab'</span>
      <span class="token punctuation">-</span> <span class="token string">'./gitlab/logs:/var/log/gitlab'</span>
      <span class="token punctuation">-</span> <span class="token string">'./gitlab/data:/var/opt/gitlab'</span>
    <span class="token key atrule">shm_size</span><span class="token punctuation">:</span> <span class="token string">'256m'</span>
    
  <span class="token key atrule">gitlab-runner</span><span class="token punctuation">:</span>
    <span class="token key atrule">image</span><span class="token punctuation">:</span> gitlab/gitlab<span class="token punctuation">-</span>runner<span class="token punctuation">:</span>latest
    <span class="token key atrule">container_name</span><span class="token punctuation">:</span> gitlab<span class="token punctuation">-</span>runner
    <span class="token key atrule">restart</span><span class="token punctuation">:</span> always
    <span class="token key atrule">depends_on</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> gitlab
    <span class="token key atrule">volumes</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> <span class="token string">'./gitlab-runner/config:/etc/gitlab-runner'</span>
      <span class="token punctuation">-</span> <span class="token string">'/var/run/docker.sock:/var/run/docker.sock'</span>
</code></pre>
<p><strong>หมายเหตุสำคัญ:</strong></p>
<ul>
<li>แทนที่ <code>gitlab.example.com</code> ด้วย IP Address หรือ Domain Name ของเซิร์ฟเวอร์</li>
<li>หากพอร์ต 22 ถูกใช้งานอยู่ ให้เปลี่ยนเป็น <code>'2222:22'</code></li>
</ul>
<hr>
<h2 id="ขั้นตอนที่-3-การเริ่มระบบ">ขั้นตอนที่ 3: การเริ่มระบบ</h2>
<h3 id="เริ่มต้น-container">3.1 เริ่มต้น Container</h3>
<p>รันคำสั่งเพื่อเริ่มต้นระบบ:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker compose up -d
</code></pre>
<p><strong>คำเตือน:</strong> กระบวนการนี้ใช้เวลาประมาณ 5-15 นาที</p>
<h3 id="ตรวจสอบสถานะ">3.2 ตรวจสอบสถานะ</h3>
<p>ตรวจสอบว่า Container ทำงานปกติ:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">ps</span>
</code></pre>
<p>คุณควรเห็น Container ทั้งสองอยู่ในสถานะ “Up”</p>
<hr>
<h2 id="ขั้นตอนที่-4-การตั้งค่าบัญชีผู้ดูแลระบบ">ขั้นตอนที่ 4: การตั้งค่าบัญชีผู้ดูแลระบบ</h2>
<h3 id="การดึงรหัสผ่านเริ่มต้น">4.1 การดึงรหัสผ่านเริ่มต้น</h3>
<p>GitLab จะสร้างรหัสผ่านชั่วคราวสำหรับบัญชี root โดยอัตโนมัติ ใช้คำสั่งนี้เพื่อดูรหัสผ่าน:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">exec</span> gitlab <span class="token function">grep</span> <span class="token string">'Password:'</span> /etc/gitlab/initial_root_password
</code></pre>
<p>ผลลัพธ์จะแสดงในรูปแบบ:</p>
<pre><code>Password: ABC-xyz-1234567890
</code></pre>
<p><strong>หมายเหตุ:</strong> บันทึกรหัสผ่านนี้ไว้ใช้งานชั่วคราว</p>
<h3 id="การเข้าสู่ระบบครั้งแรก">4.2 การเข้าสู่ระบบครั้งแรก</h3>
<ol>
<li>เปิดเว็บเบราว์เซอร์และเข้าสู่ <code>http://&lt;IP-Address-ของคุณ&gt;</code></li>
<li>กรอกข้อมูลการเข้าสู่ระบบ:
<ul>
<li>ชื่อผู้ใช้: <code>root</code></li>
<li>รหัสผ่าน: รหัสผ่านที่ได้จากขั้นตอน 4.1</li>
</ul>
</li>
<li>ระบบจะแสดงหน้าต่างให้เปลี่ยนรหัสผ่าน</li>
<li>ตั้งรหัสผ่านใหม่ที่มีความปลอดภัยสูง</li>
<li>คลิกปุ่ม “Change your password”</li>
</ol>
<hr>
<h2 id="ขั้นตอนที่-5-การลงทะเบียน-gitlab-runner">ขั้นตอนที่ 5: การลงทะเบียน GitLab Runner</h2>
<h3 id="การขอ-token-สำหรับลงทะเบียน">5.1 การขอ Token สำหรับลงทะเบียน</h3>
<ol>
<li>เข้าสู่ระบบ GitLab ด้วยบัญชี root</li>
<li>คลิกที่ไอคอนเมนู &gt; Admin Area</li>
<li>เลือกเมนู CI/CD &gt; Runners</li>
<li>บันทึก Registration URL และ Registration Token</li>
</ol>
<h3 id="การลงทะเบียน-runner">5.2 การลงทะเบียน Runner</h3>
<p>รันคำสั่งเพื่อเริ่มกระบวนการลงทะเบียน:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker <span class="token function">exec</span> -it gitlab-runner gitlab-runner register
</code></pre>
<p>กรอกข้อมูลตามที่ระบบถาม:</p>

<table>
<thead>
<tr>
<th align="left">รายการ</th>
<th align="left">ค่าที่แนะนำ</th>
<th align="left">รายละเอียด</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">GitLab instance URL</td>
<td align="left"><code>http://gitlab</code></td>
<td align="left">ใช้ชื่อ Container เพื่อการสื่อสารภายในเครือข่าย Docker</td>
</tr>
<tr>
<td align="left">Registration token</td>
<td align="left">(Token จาก Admin Area)</td>
<td align="left">คัดลอกจากหน้า Runners ในส่วน Admin</td>
</tr>
<tr>
<td align="left">Runner description</td>
<td align="left"><code>docker-executor</code></td>
<td align="left">ชื่อที่ใช้อธิบาย Runner นี้</td>
</tr>
<tr>
<td align="left">Runner tags</td>
<td align="left"><code>docker,linux,ci</code></td>
<td align="left">Tags สำหรับกำหนดว่า Job ใดจะใช้ Runner นี้</td>
</tr>
<tr>
<td align="left">Executor</td>
<td align="left"><code>docker</code></td>
<td align="left">ประเภทของ Executor ที่จะใช้รัน Job</td>
</tr>
<tr>
<td align="left">Default Docker image</td>
<td align="left"><code>alpine:latest</code></td>
<td align="left">Image พื้นฐานที่ใช้สำหรับรัน Job</td>
</tr>
</tbody>
</table><h3 id="การตรวจสอบการลงทะเบียน">5.3 การตรวจสอบการลงทะเบียน</h3>
<p>กลับไปที่หน้า Admin Area &gt; CI/CD &gt; Runners คุณจะเห็น Runner ที่เพิ่งลงทะเบียนปรากฏในรายการ</p>
<hr>
<h2 id="สรุป">สรุป</h2>
<p>ขณะนี้คุณได้ติดตั้งและกำหนดค่า GitLab CE พร้อมกับ GitLab Runner สำเร็จแล้ว ระบบพร้อมสำหรับการใช้งาน CI/CD Pipeline</p>
<h3 id="ขั้นตอนถัดไป">ขั้นตอนถัดไป</h3>
<ul>
<li>สร้าง Repository สำหรับโครงการ</li>
<li>เพิ่มไฟล์ <code>.gitlab-ci.yml</code> เพื่อกำหนด Pipeline</li>
<li>ทดสอบการทำงานของ CI/CD</li>
</ul>
<hr>
<h2 id="การแก้ไขปัญหาเบื้องต้น">การแก้ไขปัญหาเบื้องต้น</h2>
<p><strong>กรณีที่ GitLab ไม่สามารถเข้าถึงได้:</strong></p>
<ul>
<li>ตรวจสอบว่า Container ทำงานด้วย <code>sudo docker ps</code></li>
<li>ดู logs ด้วย <code>sudo docker logs gitlab</code></li>
</ul>
<p><strong>กรณีที่ Runner ไม่สามารถลงทะเบียนได้:</strong></p>
<ul>
<li>ตรวจสอบว่า URL และ Token ถูกต้อง</li>
<li>ตรวจสอบว่า Container ทั้งสองสื่อสารกันได้</li>
</ul>
<p><strong>การรีสตาร์ทระบบ:</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker compose restart
</code></pre>
<p><strong>การหยุดระบบ:</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker compose down
</code></pre>

