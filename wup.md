# Spinner

## description 

- đây có vẻ là 1 bài liên quan đến config javascript 

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
