# `DesignPattern ๐จ`

<img src="https://media.giphy.com/media/VLcvUs5uiHbCD7dHd2/giphy.gif" width="130px">

> DesignPattern ๊ฒํฅ๊ธฐ

## Study Rule

1. ์ ํด์ง ์ฃผ์ ๋ฅผ ๊ณต๋ถ/์ ๋ฆฌํ๋ค.
2. `{n}week-{name}` ๋ธ๋์น๋ฅผ ์์ฑํ์ฌ ์ ๋ฆฌ๋ ๋ด์ฉ์ Push ํ๋ค.
3. ๋งค์ฃผ ๋ชฉ์์ผ ์คํฐ๋ ์ ๊น์ง `Pull-Request` ๋ณด๋ธ๋ค.
4. ๋ชฉ์์ผ ์คํฐ๋ ์งํ ์ ํด๋น ๋ฐํ `PR` ์ ๋ฆฌ๋ทฐ๋ฅผ ์์ฑ โญ
5. ๋ฌธ์ ๋ด๋น์๋ ๋ฆฌ๋ทฐ๊ฐ ๋ชจ๋ ์๋ฃ๋ ๊ฒ์ ํ์ธํ๊ณ  `Merge` ํ๋ค.
6. ๋ฌธ์ ๋ด๋น์๋ ๋ด์ฉ์ ์ ๋ฆฌํ์ฌ GitHub์ ์ฌ๋ฆฐ๋ค. 

## ๋ฌธ์ ์๋ก๋ ๋ฐ ๋ฆฌ๋ทฐ Rule

1. ๊ฐ์ธ ๊ณต๋ถ๋ ์ฃผ์ฐจ๋ณ ํด๋์ ์์ ๋กญ๊ฒ ์ ๋ฆฌํด์ ๊ณต์ ํ๋ค.
2. ๋ฐํ๋ Notion, Blog, PPT, Github ๋ฑ ํธํ ๋ฐฉ์์ผ๋ก ์งํํ๋ค.
3. ๋ฆฌ๋ทฐ๋ ์ ํด์ง ๊ท๊ฒฉ ์์ด ์์ ๋กญ๊ฒ ์์ฑํ๋ค.
   - ๊ถ๊ธํ ์ฌํญ
   - ๋ ์กฐ์ฌํ๋ฉด ์ข์ ๊ฒ ๊ฐ์ ๋ด์ฉ
   - ํนํ ์ข์๋ ๋ถ๋ถ ๋ฑ
4. ๋ฆฌ๋ทฐ๊ฐ ์๋ฃ๋๋ฉด `viewed` ์ฒดํฌ ๋ฐ `Approve` ๋ก ์๋ฃ๋ฅผ ์๋ฆฐ๋ค.
5. `Merge` ๋ ๋ฌธ์ ๋ด๋น์๊ฐ ๋ชจ๋  Reviewers ์๋ฃ๋ฅผ ํ์ธํ๋ฉด ์งํํ๋ค.
6. ๋ฌธ์ ๋ด๋น์๋ ์ฃผ์ฐจ๋ณ ํด๋์ README.md์ ์ ๋ฆฌํ์ฌ ์์ฑํ๋ค.

## ๋ชฉ์ฐจ ๐

<table>
    <thead>
        <tr>
            <th>์น์</th>
            <th>์ฃผ์ฐจ</th>
            <th>๋ฌธ์ ๋ด๋น์</th>
            <th>์ฃผ์ </th>
            <th>๋ฐํ ์์</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=6>์น์ 1 : ๊ฐ์ฒด ์์ฑ</td>
            <td>1์ฃผ์ฐจ</td>
            <td>์์ฑ</td>
            <td>์ฑ๊ธํค ํจํด</td>
            <td>์์ค-์ฌ์ฐ-์ฐฝ์ญ-์์ฑ</td>
        </tr>
        <tr>
            <td rowspan=2>2์ฃผ์ฐจ</td>
            <td rowspan=2>์ฐฝ์ญ</td>
            <td>ํฉํ ๋ฆฌ ๋ฉ์๋ ํจํด</td>
            <td>์ฌ์ฐ-์์ค</td>
        </tr>
        <tr>
            <td>์ถ์ ํฉํ ๋ฆฌ ํจํด</td>
            <td>์์ฑ-์ฐฝ์ญ</td>
        </tr>
        <tr>
            <td rowspan=2>3์ฃผ์ฐจ</td>
            <td rowspan=2>์ฐฝํ</td>
            <td>๋น๋ ํจํด</td>
            <td>์์ค-์์ฑ</td>
        </tr>
        <tr>
            <td>ํ๋กํ ํ์ ํจํด</td>
            <td>์ฐฝ์ญ-์ฌ์ฐ</td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <td rowspan=7>์น์ 2 : ๊ตฌ์กฐ</td>
            <td rowspan=2>4์ฃผ์ฐจ</td>
            <td rowspan=2>์ฌ์ฐ</td>
            <td>์ด๋ํฐ ํจํด</td>
            <td>์ฐฝํ-์์ค</td>
        </tr>
        <tr>
            <td>๋ธ๋ฆฟ์ง ํจํด</td>
            <td>์์ฑ-์ฐฝ์ญ</td>
        </tr>
        <tr>
            <td rowspan=2>5์ฃผ์ฐจ</td>
            <td rowspan=2>์ฐฝ์ญ</td>
            <td>์ปดํฌ์ง ํจํด</td>
            <td>์ฌ์ฐ-์ฐฝํ</td>
        </tr>
        <tr>
            <td>๋ฐ์ฝ๋ ์ดํฐ ํจํด</td>
            <td>์์ค-์์ฑ</td>
        </tr>
        <tr>
            <td rowspan=2>6์ฃผ์ฐจ</td>
            <td rowspan=2>์์ฑ</td>
            <td>ํผ์ฌ๋ ํจํด</td>
            <td>์ฐฝ์ญ-์ฌ์ฐ</td>
        </tr>
        <tr>
            <td>ํ๋ผ์ด์จ์ดํธ ํจํด</td>
            <td>์ฐฝํ-์์ค</td>
        </tr>
        <tr>
            <td>7์ฃผ์ฐจ</td>
            <td>์์ค</td>
            <td>ํ๋ก์ ํจํด</td>
            <td>์์ฑ-์ฐฝ์ญ</td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <td rowspan=11>์น์ 3 : ํ๋</td>
            <td>7์ฃผ์ฐจ</td>
            <td>์์ค</td>
            <td>์ฑ์ ์ฐ์ ํจํด</td>
            <td>์ฌ์ฐ-์ฐฝํ</td>
        </tr>
        <tr>
            <td rowspan=2>8์ฃผ์ฐจ</td>
            <td rowspan=2>์ฐฝํ</td>
            <td>์ปค๋งจ๋ ํจํด</td>
            <td>์์ค-์์ฑ</td>
        </tr>
        <tr>
            <td>์ธํฐํ๋ฆฌํฐ ํจํด</td>
            <td>์ฐฝ์ญ-์ฌ์ฐ</td>
        </tr>
        <tr>
            <td rowspan=2>9์ฃผ์ฐจ</td>
            <td rowspan=2>์ฌ์ฐ</td>
            <td>์ดํฐ๋ ์ดํฐ ํจํด</td>
            <td>์ฐฝํ-์์ค</td>
        </tr>
        <tr>
            <td>์ค์ฌ์ ํจํด</td>
            <td>์์ฑ-์ฐฝ์ญ</td>
        </tr>
        <tr>
            <td rowspan=2>10์ฃผ์ฐจ</td>
            <td rowspan=2>์ฐฝ์ญ</td>
            <td>๋ฉ๋ฉํ  ํจํด</td>
            <td>์ฌ์ฐ-์ฐฝํ</td>
        </tr>
        <tr>
            <td>์ต์ ๋ฒ ํจํด</td>
            <td>์์ค-์์ฑ</td>
        </tr>
        <tr>
            <td rowspan=2>11์ฃผ์ฐจ</td>
            <td rowspan=2>์์ฑ</td>
            <td>์ํ ํจํด</td>
            <td>์ฐฝ์ญ-์ฌ์ฐ</td>
        </tr>
        <tr>
            <td>์ ๋ต ํจํด</td>
            <td>์ฐฝํ-์์ค</td>
        </tr>
        <tr>
            <td rowspan=2>12์ฃผ์ฐจ</td>
            <td rowspan=2>์ฐฝํ</td>
            <td>ํํ๋ฆฟ ๋ฉ์๋ ํจํด</td>
            <td>์์ค-์์ฑ</td>
        </tr>
        <tr>
            <td>๋น์งํฐ ํจํด</td>
            <td>์ฐฝ์ญ-์ฌ์ฐ</td>
        </tr>
    </tbody>

</table>

- Reference : [์ธํ๋ฐ ๋ฐฑ๊ธฐ์ ๋ ๊ฐ์](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)

## Collaborator

| <img src="https://avatars.githubusercontent.com/u/42997924?v=4" width=100> | <img src="https://avatars.githubusercontent.com/u/17822723?v=4" width=100> | <img src="https://avatars.githubusercontent.com/u/52964858?v=4" width=100> | <img src="https://avatars.githubusercontent.com/u/32676275?v=4" width=100> | <img src="https://avatars.githubusercontent.com/u/79291114?v=4" width=100>  |
| :---: | :---: | :---: | :---: | :---: |
| ์์ค([@jun108059](https://github.com/jun108059)) | ์ฐฝ์ญ([@ventulus95](https://github.com/ventulus95)) | ์์ฑ([@gowoonsori](https://github.com/gowoonsori)) | ์ฌ์ฐ([@CJW23](https://github.com/CJW23)) | ์ฐฝํ([@dev-splin](https://github.com/dev-splin)) |
