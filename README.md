# Next.js Application Dockerize করা

এই ডকুমেন্টে একটি সাধারণ Next.js অ্যাপ্লিকেশন তৈরি করে সেটাকে Docker-এর মাধ্যমে containerize করার পুরো প্রক্রিয়া ধাপে ধাপে দেখানো হয়েছে।

---

## ধাপ ১: Next.js Project Setup

প্রথমে PowerShell থেকে নতুন একটি Next.js project তৈরি করতে হবে।

#### PowerShell open করে command দাও:

```bash
npx create-next-app@latest nextjs-project-dockerize
```

`create-next-app` কিছু configuration সম্পর্কে প্রশ্ন করবে। প্রয়োজন অনুযায়ী option নির্বাচন করে project তৈরি করতে হবে।

Project তৈরি হয়ে গেলে project folder-এ ঢুকতে হবে:

```bash
cd nextjs-project-dockerize
```

তারপর project-টি VS Code-এ open করতে পারো:

```bash
code .
```

---

## ধাপ ২: Next.js Application Run করে দেখা

Dockerize করার আগে local machine-এ Next.js application ঠিকভাবে কাজ করছে কিনা পরীক্ষা করে নেওয়া ভালো।

প্রথমে development server চালাও:

```bash
npm run dev
```

এরপর browser-এ যাও:

```text
http://localhost:3000
```

Next.js-এর default page দেখতে পেলে বুঝবে project ঠিকভাবে তৈরি হয়েছে।

Server বন্ধ করতে terminal-এ:

```text
Ctrl + C
```

---

## ধাপ ৩: VS Code এ Docker Extension Install করা

VS Code editor-এর **Extensions** button-এ click করো।

Search করো:

```text
Docker
```
<img width="1554" height="230" alt="image" src="https://github.com/user-attachments/assets/05f5d142-af97-4ae1-8b64-62b1c1109a79" />

তারপর Microsoft-এর **Docker** extension install করো।

এই extension ব্যবহার করে VS Code থেকেই Dockerfile, images এবং containers নিয়ে কাজ করা সহজ হয়।

---

## ধাপ ৪: Project Root এ Dockerfile তৈরি করা

এখন Next.js project-এর root directory-তে একটি file তৈরি করতে হবে।

File-এর নাম হবে:

```text
Dockerfile
```

> **Important:** `Dockerfile`-এর কোনো extension থাকবে না।
> যেমন: `Dockerfile.txt` দেওয়া যাবে না।

Project structure মোটামুটি এমন হবে:

```text
nextjs-project-dockerize/
│
├── app/
├── public/
├── node_modules/
├── package.json
├── package-lock.json
├── next.config.ts
└── Dockerfile
```

---

## ধাপ ৫: Dockerfile তৈরি করা

এখন `Dockerfile`-এর মধ্যে নিচের code লিখতে হবে:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

---

## Dockerfile এর প্রতিটি instruction-এর ব্যাখ্যা

| Instruction             | কাজ                                                                      |
| ----------------------- | ------------------------------------------------------------------------ |
| `FROM node:20-alpine`   | Node.js 20 Alpine Linux image-কে base image হিসেবে ব্যবহার করা হচ্ছে     |
| `WORKDIR /app`          | Container-এর ভিতরে `/app` নামে working directory তৈরি/নির্ধারণ করা হচ্ছে |
| `COPY package*.json ./` | `package.json` এবং `package-lock.json` container-এ copy করা হচ্ছে        |
| `RUN npm install`       | Project-এর dependencies install করা হচ্ছে                                |
| `COPY . .`              | Project-এর বাকি সব file container-এ copy করা হচ্ছে                       |
| `RUN npm run build`     | Next.js application-এর production build তৈরি করা হচ্ছে                   |
| `EXPOSE 3000`           | Application যে port-এ চলবে, সেটি `3000` হিসেবে নির্দেশ করা হচ্ছে         |
| `CMD ["npm", "start"]`  | Container চালু হলে Next.js production server start হবে                   |

---

## ধাপ ৬: `.dockerignore` তৈরি করা

Docker image build করার সময় কিছু file/container-এর জন্য অপ্রয়োজনীয় directory copy না করাই ভালো।

তাই project root-এ আরেকটি file তৈরি করো:

```text
.dockerignore
```

এর মধ্যে লিখো:

```text
node_modules
.next
.git
.gitignore
Dockerfile
.dockerignore
npm-debug.log
```

### `.dockerignore` কেন ব্যবহার করা হয়?

`.dockerignore` Docker-কে বলে দেয় কোন কোন file বা folder build context-এর সময় container-এ copy না করতে হবে।

যেমন:

```text
node_modules
```

local machine-এর `node_modules` container-এ copy করার দরকার নেই, কারণ Dockerfile-এর মধ্যে আমরা নতুন করে:

```dockerfile
RUN npm install
```

চালাচ্ছি।

একইভাবে `.next` folder-ও copy করার দরকার নেই, কারণ Dockerfile-এর মধ্যে:

```dockerfile
RUN npm run build
```

চালিয়ে নতুন করে production build তৈরি করা হচ্ছে।

---

## ধাপ ৭: Docker Image Build করা

এখন project root directory থেকে নিচের command চালাতে হবে:

```bash
docker build -t nextjs-project-dockerize .
```

এখানে:

* `docker build` → Docker image build করার command
* `-t nextjs-project-dockerize` → image-এর নাম/tag দেওয়া হচ্ছে
* `.` → current directory-কে build context হিসেবে ব্যবহার করা হচ্ছে

Dockerfile এবং `.dockerignore` যেহেতু current directory-তে আছে, তাই `.` দেওয়া হয়েছে।

---

## ধাপ ৮: Docker Image Check করা

Build হওয়া image দেখতে নিচের command চালাও:

```bash
docker images
```

এখানে এমন একটি image দেখতে পাবে:

```text
REPOSITORY                  TAG       IMAGE ID       SIZE
nextjs-project-dockerize   latest    xxxxxxxxxxxx   ...
```

এর অর্থ Next.js application-এর Docker image successfully তৈরি হয়েছে।

---

## ধাপ ৯: Docker Container Run করা

এখন image থেকে একটি container চালাতে হবে:

```bash
docker run -p 3000:3000 nextjs-project-dockerize
```

এখানে:

```text
3000:3000
```

এর অর্থ:

```text
Host Machine Port : Container Port
```

অর্থাৎ:

```text
localhost:3000  →  Container:3000
```

---

## ধাপ ১০: Browser থেকে Application দেখা

Container successfully run করার পর browser open করে যাও:

```text
http://localhost:3000
```

Next.js application-এর page দেখতে পাবে।

অর্থাৎ এখন Next.js application সরাসরি local machine-এ `npm run dev` দিয়ে নয়, বরং **Docker container-এর ভিতরে production mode-এ চলছে।**

---

## ধাপ ১১: Running Container Check করা

কোন কোন container বর্তমানে চলছে তা দেখতে:

```bash
docker ps
```

এখানে তোমার Next.js container দেখতে পাবে।

Container বন্ধ করতে:

```bash
docker stop <container-id>
```

অথবা container-এর নাম ব্যবহার করেও stop করা যায়:

```bash
docker stop <container-name>
```

---
