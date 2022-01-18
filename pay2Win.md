+++
title = "BuckEye: Pay2win"
date = "2021-10-23"
aliases = ["Pay2win"]
[ author ]
  name = "Lucas Sass Rósinberg"
+++

# Pay2win

Opening the URL, we immediately get loads of popups. It detects if we try to open devtools and redirects to a rick roll clip.

We can disable JS in the browser to avoid the popups, allowing us to open devtools. Inspecting the source, we find `main.js`, which among others contain the function

```javascript=
function plantFlag () {
  const ciphertext = [234, 240, 234, 252, 214, 236, 140, 247, 173, 191, 158, 132, 56, 4, 32, 73, 235, 193, 233, 152, 125, 19, 19, 237, 186, 131, 98, 52, 186, 143, 127, 43, 226, 233, 126, 15, 225, 171, 85, 55, 173, 123, 21, 147, 97, 21, 237, 11, 254, 129, 2, 131, 101, 63, 149, 61]
  const plaintext = ciphertext.map((x, i) => ((i * i) % 256) ^ x ^ 0x99)

  const flagElement = document.querySelector('#flag')
  plaintext.map((x, i) => {
    const span = document.createElement('span')
    span.classList.add(`flag-char-${i}`)
    span.textContent = String.fromCharCode(x)
    flagElement.appendChild(span)
    return span
  })

  const flagOverlay = document.querySelector('#flag-overlay')
  flagOverlay.addEventListener('mouseover', async () => {
    await swal(flagAlert)
  })
}
```

Running just the first two lines, we get a list of numbers within ASCII range. We can convert it to a string with

```javascript=
plaintet.map(x => String.fromCharCode(x)).join("")
```

but this gives us

> shwl_l1_twcd14}1ry4ht3neck_t3_bs{1c_hkh_tsh3he03gy_3l_hu

which looks like a scrambled flag with the right characters. The rest of `plantFlag()` inserts the plaintext in the element with id `flag`. This field is overlayed, so it's not immediately visible. The script then adds a listener to create a paywall popup when the overlay is moused over (assuming JS is enabled). But we can just remove the overlay in devtools, allowing us to see the flag:

> buckeye{h0ly_sh1t_wh4t_th3_h3ck_1s_th1s_w31rd_ch4ll3ng3}

Why was the order of the characters different on the page? Because each character was in its own `<span>` element with its own id, and each had a different flex-order, swapping them into their correct spots.
