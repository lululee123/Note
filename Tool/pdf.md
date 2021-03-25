# PDF

這邊紀錄一下遇到需要下載 PDF 的各種處理方式

[TOC]


方法 1: 頁面直接輸出 PDF 
---

#### 使用套件
- [html2pdf](https://ekoopmans.github.io/html2pdf.js/)

#### 使用情境

    畫面單純可直接輸出

#### Example code
```javascript= 
/**
 * @param {DOM element} e
 */

const downloadPdf = e => {
    const opt = {
        margin: 0,
        filename: 'survey.pdf',
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: { scale: 2, useCORS: true },
        jsPDF: { unit: 'in', format: 'letter', orientation: 'portrait' },
        pagebreak: { mode: ['avoid-all', 'css', 'legacy'] },
    };

    html2pdf()
    .from(e)
    .set(opt)
    .save();
};
```

方法 2: 複雜頁面輸出 PDF 
---
#### 使用套件 
- [html2canvas](https://html2canvas.hertzen.com/) 
- [jsPDF](https://github.com/MrRio/jsPDF)

#### 使用情境
    畫面過於複雜，須先將畫面拆解成塊，拚成 PDF 後再做輸出

#### Example code
```javascript=
/**
 * @param {DOM element} e
 */

const downloadPdf = async (e, fileName, onePage = false) => {
    // 傳入的 DOM element array
    const checkArray = e.filter(item => item.current);
    const canvasArray = [];

    // 將 HTML 轉換成 canvas
    for (let i = 0; i < checkArray.length; i += 1) {
        window.scrollTo(0, 0);
        await html2canvas(checkArray[i].current, {
            scrollX: 0,
            scrollY: 0,
        }).then(canvas => {
            canvasArray.push({
                width: canvas.width,
                height: canvas.height,
                widthInPDF: 400,
                heightInPDF: (400 / canvas.width) * canvas.height,
                image: canvas.toDataURL(),
            });
        });
    }

    // 將 canvas 拼入 PDF，如果無法輸出成一頁，需計算換頁
    if (onePage) {
        const pageHeight = canvasArray.reduce((acc, current) => acc + current.heightInPDF, 0);
        const pdf = new jsPDF('p', 'pt', [590, pageHeight + 40 + (20 * canvasArray.length)], true);
        let position = 20;

        canvasArray.forEach(canvasData => {
            const { image, widthInPDF, heightInPDF } = canvasData;

            pdf.addImage(image, 'JPEG', 95, position, widthInPDF, heightInPDF, undefined, 'FAST');

            // 下一張圖片位置
            position = position + heightInPDF + 20;
        });

        pdf.save('PDF.pdf');
    } else {
        const pdf = new jsPDF('p', 'pt', 'a4');
        let position = 20;
        const pageHeight = 822;
        let leftHeight = 0;
        let long = false;

        canvasArray.forEach(canvasData => {
            const { widthInPDF, heightInPDF, image } = canvasData;
            leftHeight = heightInPDF;

            if (heightInPDF > pageHeight) {
                long = true;
                pdf.addPage();
                position = 20;
                while (leftHeight > 0) {
                    pdf.addImage(image, 'JPEG', 95, position, widthInPDF, heightInPDF, undefined, 'FAST');
                    leftHeight -= 842;
                    position -= 842;
                    if (leftHeight > 0) {
                        pdf.addPage();
                    }
                }
            } else {
                if (position + heightInPDF > pageHeight || long) {
                    long = false;
                    pdf.addPage();
                    position = 20;
                }

                pdf.addImage(image, 'JPEG', 95, position, widthInPDF, heightInPDF, undefined, 'FAST');

                // 下一張圖片位置
                position = position + heightInPDF + 20;
            }
        });

        pdf.save(`${fileName}.pdf`);
    }
};
```

方法 3: 下載後端回傳的 base64 PDF 檔案
---

#### Example code
```javascript=
/**
 * @param {pdf file} pdf
 */

const downloadPdf = pdf => {
    const linkSource = `data:application/pdf;base64, ${pdf}`;
    const downloadLink = document.createElement('a');
    downloadLink.href = linkSource;
    downloadLink.download = fileName;
    downloadLink.click();
}
```

#### 坑
- 上述方法在 iOS 14 的 Safari 無法使用，[這邊](https://25sprout.slack.com/archives/G07AKRYLQ/p1615803197012000)是當時的解法紀錄

> Ref: https://stackoverflow.com/questions/64152225/is-there-a-workaround-to-create-a-custom-download-link-for-safari-14



注意事項
---

- svg 圖片在輸出時，需要設定  `width` `height`
- svg 圖片在輸出時，吃不到使用 css 設定 path fill 的顏色，需要直接設定在 svg 檔案上

###### tags: `PDF` `html2pdf` `html2canvas` `jsPDF`
