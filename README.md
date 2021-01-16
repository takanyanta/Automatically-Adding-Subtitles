# Automatically-Adding-Subtitles

## Purpose for coding
* To see foreign movie with English(or other language except Japanese) subtitles.

## Used tequnique
|  Used Technique  |  Used Libraries  | Used Tools |
| ---- | ---- | ---- |
|  OCR  |  pyocr  | Tesseract |
|  Translation  |  googletrans  | - |
| General | openCV, PIL | - |

## Process outline
1. Extract the frame from the movie

```python
    #extract frame at determined point 
    cap.set(cv2.CAP_PROP_POS_FRAMES, num)
```

![Extract the frame](https://github.com/takanyanta/Automatically-Adding-Subtitles/blob/main/ResultPic/Before_000.png "process1")

2. Extract the area from the frame

```python
        frame_subtitles = frame[300:frame.shape[0], 100:600]
```

![Extract the frame](https://github.com/takanyanta/Automatically-Adding-Subtitles/blob/main/ResultPic/add_000.png "process1")

3. For OCR reading, preprocessing frame data

* Results of translation

```python
        frame_subtitles_gray = cv2.cvtColor(frame_subtitles, cv2.COLOR_BGR2GRAY)
        #plt.imshow(frame_subtitles_gray)
        #plt.show()
        threshold = 225
        ret, frame_subtitles_binary = cv2.threshold(frame_subtitles_gray, threshold, 255, cv2.THRESH_BINARY)
        #plt.imshow(frame_subtitles_binary)
        #plt.show()
        frame_subtitles_binary_reverse = cv2.bitwise_not(frame_subtitles_binary)
        #plt.imshow(frame_subtitles_binary_reverse)
        #plt.show()
        frame_subtitles_binary_reverse_image = Image.fromarray(frame_subtitles_binary_reverse)
```

4. OCR and translate them

```python
def OCR_read(PIL_data):
    tools = pyocr.get_available_tools()
    if len(tools) == 0:
        print("No OCR tool found")
        sys.exit(1)
    tool = tools[0]
    txt = tool.image_to_string( #define language, options
            PIL_data,
            lang='eng',
            builder=pyocr.builders.TextBuilder(tesseract_layout=6)
            )
    txt1 = txt.replace('\n', " ")#Replace line break with space

    translator = Translator()
    translated = translator.translate(txt1, dest="ja")
    
    return txt1, translated.text
```

5. return the result of 4. to original frame

```python
        try:
            en, ja = OCR_read(frame_subtitles_binary_reverse_image)

            #Define where to write Japanese subtitles
            size_ = 14
            start_ = ( (width-size_*len(ja))*0.5 )
            height_ = (300)

            #Wrinting below
            font = ImageFont.truetype("ipag.ttf", size_)
            frame_rgb_ = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            pil_image_ = Image.fromarray(frame_rgb_)
            draw_ = ImageDraw.Draw(pil_image_)
            draw_.text((start_, height_), ja, font=font)

            #change cv2 format
            rgb_image_ = cv2.cvtColor(np.array(pil_image_), cv2.COLOR_RGB2BGR)
            out.write(rgb_image_)
            #plt.imshow(rgb_image_)
            #plt.show()
            res_.append([num, en, ja])
            cv2.imwrite(r"ResultPic\After_{:03d}.png".format(num), rgb_image_)
        except TypeError:
            pass
```

![Extract the frame](https://github.com/takanyanta/Automatically-Adding-Subtitles/blob/main/ResultPic/After_000.png "process1")

### Result Review

* Translation Result
I feel it is not so bad.

|  Sec  |  English  | Japanese |
| ---- | ---- | ---- |
|  0  |  -You brat, you don't know anything! -That's right!  | -ガキ、何も知らない！-そのとおり！ |
|  3  |  If I'm your successor, will I have  | 私があなたの後継者なら、 |
| 6 | the authority to appoint new CEOs for our subsidiaries? | 子会社の新しいCEOを任命する権限はありますか？ |
|  9  |  That's great.  | それは素晴らしいことです。 |
|  12  |  I always thought that a few of the CEOs are underqualified.  | 私はいつも、CEOの何人かは資格が不足していると思っていました。 |
| 15 | For instance, those who tend to clash with their employees, PIL | たとえば、従業員と衝突する傾向がある人 |
|  18  |  For instance, those who tend to clash with their employees  | たとえば、従業員と衝突する傾向がある人 |
|  21  |  or have caused the company great damage due to their own ignorance.  | または彼ら自身の無知のために会社に大きな損害を与えました。 |
| 24 | or have caused the company great damage due to their own ignorance., PIL | または彼ら自身の無知のために会社に大きな損害を与えました。 |
|  27  |  We should fire such CEGs without any hesitation.  | そのようなCEGをためらうことなく発射する必要があります。 |
|  30  |  all  | すべて |
| 33 | Give me some time though., PIL | でも少し時間をください。 |
|  36  |  My company recently launched an extreme-sports apparel line,  | 私の会社は最近、エクストリームスポーツのアパレルラインを立ち上げました。 |
|  39  |  so it's grown even bigger.  | だからそれはさらに大きくなりました。 |
| 42 | T'll look for a good management specialist. a., PIL | 良い管理スペシャリストを探します。a。 |

### Conclusion
* Translation accuracy is not so bad.
* Computational speed is very slow.(This is the challenges to overcome for commercail use)
* googletranslate sometimes returns different result with same sentense.(This problem is seems to be solved by using NLP technique, such as "cos similarity" when OCR results is same as the previpous one)
