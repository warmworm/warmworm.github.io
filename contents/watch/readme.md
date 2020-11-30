시계 프로그램
============

2002년 11월 15일에 자바 애플릿으로 만든 시계 프로그램을 JavaScript로 컨버팅.

아래는 자바 애플릿 버전의 원래 소스코드

```Java
/*
 * 손목시계 스타일의 애플릿 v(ㅡ,.ㅡ)y-~
 *
 *  +===========================+
 *  |                           |
 *  |   2002-11-15    금   오전  |
 *  | ------------------------- |
 *  |      12:24      45        |
 *  |                           |
 *  +===========================+
 *
 */

import java.awt.*;
import java.awt.image.*;
import java.applet.*;
import java.util.*;

public class Watch extends Applet implements Runnable {
    Image i_watch = null;                   // 시계 그림
    Image i_watchfont_orig = null;          // 폰트 소스 이미지
    Image i_time_back = null;               // 숫자 출력부 배경 그림
    Image[] i_font_ampm = new Image[2];     // [0]=오전, [1]=오후
    Image[] i_font_hour = new Image[11];    // 폰트: 시,분
    Image[] i_font_second = new Image[11];  // 폰트: 초
    Image[] i_font_date = new Image[11];    // 폰트: 날짜
    Image[] i_font_yoil = new Image[11];    // 폰트: 요일
    Image i_backScreen = null;              // Back Screen
    Graphics bsGC = null;                   // Back Screen Graphics Context
    Calendar cal = null;
    Thread t = null;
    boolean blink_flag;                     // true=':'출력, false=':'를 출력하지 않음

    //====================================================================

    public void init() {
        // Back Screen을 만든다
        i_backScreen = createImage(200, 200);
        bsGC = i_backScreen.getGraphics();

        // 애플릿에서 사용되는 이미지들을 로딩한다
        MediaTracker mt = new MediaTracker(this);
        i_watch = getImage(getCodeBase(), "watch.gif");
        i_watchfont_orig = getImage(getCodeBase(), "watchfont.gif");
        mt.addImage(i_watch, 0);
        mt.addImage(i_watchfont_orig, 0);

        ImageProducer p = i_watchfont_orig.getSource();

        for (int i=0; i<11; i++) {
            i_font_hour[i] = cropImage((i*19+1), 15, 18, 31, p);    // 시, 분 폰트
            i_font_second[i] = cropImage((i*12+1), 70, 11, 19, p);  // 초 폰트
            i_font_date[i] = cropImage((i*8+1), 110, 7, 13, p);     // 날짜 폰트
            i_font_yoil[i] = cropImage((i*10+1), 145, 9, 11, p);    // 요일 폰트

            mt.addImage(i_font_hour[i], 0);
            mt.addImage(i_font_second[i], 0);
            mt.addImage(i_font_date[i], 0);
            mt.addImage(i_font_yoil[i], 0);
        }

        i_time_back = cropImage(170, 70, 117, 55, p);   // 숫자출력부 배경
        i_font_ampm[0] = cropImage(230, 15, 11, 7, p);  // '오전'
        i_font_ampm[1] = cropImage(242, 15, 11, 7, p);  // '오후'

        mt.addImage(i_time_back, 0);
        mt.addImage(i_font_ampm[0], 0);
        mt.addImage(i_font_ampm[1], 0);

        try {mt.waitForAll();} catch (InterruptedException ie) {}
        while ((mt.statusAll(true) & MediaTracker.COMPLETE) == 0) {}
    }

    //====================================================================

    public void start() {
        bsGC.drawImage(i_watch, 0, 0, null);    // 시계 바탕 그림
        blink_flag = true;                      // true=':' 출력, false=':' 출력하지 않음

        if (t == null) {
            t = new Thread(this);
            t.start();
        }
    }

    //====================================================================

    public void run() {
        long start_time;
        long delay;

        while (t != null) {
            // 시간 출력
            start_time = System.currentTimeMillis();
            drawTime();

            // 한 프레임당 할당된 시간에.. 그 프레임을 처리하는데 걸린
            // 시간을 빼서.. delay할 시간을 구한다.
            delay = 1000 / 4;           // 초당 4 프레임 출력
            delay -= System.currentTimeMillis() - start_time;
            try {Thread.sleep(Math.max(10, delay));} catch (Exception e) {}
        }
    }

    //====================================================================

    public void paint(Graphics g) {
        g.drawImage(i_backScreen, 0, 0, null);
    }

    //====================================================================

    public void update(Graphics g) {
        paint(g);
    }

    //====================================================================

    public void stop() {
        if (t != null) t = null;
    }

    //====================================================================

    public void drawTime() {
        cal = new GregorianCalendar();

        int jarisoo;        // 자릿수
        int x;              // X 좌표: 이미지 출력에 사용
        int y;              // Y 좌표:        "

        int year = cal.get(Calendar.YEAR);              // 년
        int month = cal.get(Calendar.MONTH) + 1;        // 월: 1부터 시작하도록 만든다
        int day = cal.get(Calendar.DATE);               // 일
        int yoil = cal.get(Calendar.DAY_OF_WEEK) - 1;   // 요일: 0부터 시작하도록 만든다
        int ampm = cal.get(Calendar.AM_PM);             // 오전, 오후 여부..
        int hour = cal.get(Calendar.HOUR);              // 시
        int minute = cal.get(Calendar.MINUTE);          // 분
        int second = cal.get(Calendar.SECOND);          // 초

        if (hour == 0) hour = 12;                       // '0'시는 '12'시로 출력
        bsGC.drawImage(i_time_back, 38, 66, this);      // 배경을 지운다.

        // '년도' 출력
        x = 42;
        y = 69;
        jarisoo = 1000;

        for (int i=0; i<4; i++) {
            bsGC.drawImage(i_font_date[year/jarisoo], x, y, this);
            year %= jarisoo;        // 출력하고 난 나머지 숫자만 남긴다.
            jarisoo /= 10;          //  자릿수를 낮춘다..
            x += 8;
        }
        bsGC.drawImage(i_font_date[10], x, y, this);    // -
        x += 8;

        // '월' 출력
        if (month >= 10) {
            bsGC.drawImage(i_font_date[1], x, y, this);
            month %= 10;
        }

        x += 8;
        bsGC.drawImage(i_font_date[month], x, y, this);     // '월'의 뒷자리
        x += 8;
        bsGC.drawImage(i_font_date[10], x, y, this);        // -
        x += 8;

        // '일' 출력
        if (day >= 10) {
            bsGC.drawImage(i_font_date[day/10], x, y, this);
            day %= 10;
        }

        x += 8;
        bsGC.drawImage(i_font_date[day], x, y, this);       // '일'의 뒷자리

        // '요일' & '오전/오후' 출력
        bsGC.drawImage(i_font_yoil[yoil], 126, 71, this);
        bsGC.drawImage(i_font_ampm[ampm], 142, 75, this);

        // '시' 출력
        x = 29;
        y = 89;

        if (hour >= 10) {
            bsGC.drawImage(i_font_hour[1], x, y, this);
            hour %= 10;
        }

        x += 19;
        bsGC.drawImage(i_font_hour[hour], x, y, this);      // '시'의 뒷자리
        x += 19;

        // ':' 출력
        if (blink_flag == true) {
            bsGC.drawImage(i_font_hour[10], x, y, this);
            blink_flag = false;
        } else {
            blink_flag = true;
        }
        x += 19;

        // '분' 출력
        bsGC.drawImage(i_font_hour[minute/10], x, y, this);
        minute %= 10;
        x += 19;
        bsGC.drawImage(i_font_hour[minute], x, y, this);    // '시'의 뒷자리

        // '초' 출력
        bsGC.drawImage(i_font_second[second/10], 128, 101, this);
        second %= 10;
        bsGC.drawImage(i_font_second[second], 140, 101, this);
        repaint();
    }

    //====================================================================

    public Image cropImage(int x, int y, int width, int height, ImageProducer ip) {
        ImageFilter f = new CropImageFilter(x, y, width, height);
        return createImage(new FilteredImageSource(ip, f));
    }
}
```