---
title: OpenCvSharp 2.x -> 4.x 버전 변경 (1)
---

<h1>OpenCvSharp 버전 변경</h1>

예전에 외주를 맡긴 적이 있었는데 그때 소프트웨어에 사용된 OpenCvSharp 버전이 2.x 버전이었다. 이 버전은 더 이상 지원을 해주지 않을 뿐더러 메모리 관리 등 4.x가 더 뛰어나기 때문에 언젠간 바꿔야 했을텐데 이번 UI 수정을 맡은 김에 이것까지 진행하게 됐다.


난 OpenCvSharp을 4.x 버전으로 처음 접해보았고 그 안에서도 어떠한 함수가 있는지, 또 어떤 역할을 하는지 잘 모른다. 그래서 최악인 영어 실력으로 Stack OverFlow를 뒤져가며 나름 열심히 찾아봤다.

바뀐 코드가 워낙 많기 때문에 나눠서 글을 쓸 것인데 첫 글은 4.x 버전의 메모리 관리에 대해서 쓰고 싶다.

내가 생각하기에 제일 체감되는 부분은 **메모리 관리 부분**이기 때문.

이 소프트웨어에서는 연결된 카메라를 화면에 보여주기 위해 **PictureBox라는** Window Forms에 있는 도구를 사용하는데
코드를 바로 보자면

```
// OpenCvSharp 2.x

// 카메라에 송출을 할 때
 private void Timer_Tick(object sender, EventArgs e)
    {
        using (Mat frame = new Mat())
        {
            if (capture.Read(frame))
            {
                pictureBox.Image = BitmapConverter.ToBitmap(frame);
            }
        }
    }
		
	// WinForms가 종료될 때
	protected override void OnFormClosing(FormClosingEventArgs e)
    {
        timer.Stop();
        capture.Release();
        capture.Dispose();
        base.OnFormClosing(e);
    }
```

<br>

```
// OpenCvSharp 4.x

// 카메라에 송출을 할 때
 private void Timer_Tick(object sender, EventArgs e)
    {
        if (capture.IsOpened())
        {
            capture.Read(frame);
            if (!frame.Empty())
            {
                bitmap?.Dispose();
                bitmap = BitmapConverter.ToBitmap(frame);
                pictureBox.Image = bitmap;
            }
        }
    }
// WinForms가 종료될 때
protected override void OnFormClosing(FormClosingEventArgs e)
    {
        timer.Stop();
        capture.Release();
        capture.Dispose();
        frame.Dispose();
        bitmap?.Dispose();
        base.OnFormClosing(e);
    }
```

<br>

메모리 관리 측면에서 바라보자면

	•	bitmap?.Dispose()를 추가하여 이전 Bitmap을 명시적으로 해제 → 메모리 누수 방지
	•	frame.Dispose() 호출하여 Mat 객체를 정리

안정성 측면에서 바라보자면

	•	IsOpened()로 카메라가 정상적으로 열렸는지 확인
	•	frame.Empty() 체크하여 빈 프레임 처리
	
	
	
이러한 이유로 4.x 버전을 쓸 수 밖에 없다.
아무래도 카메라는 화면을 계속 불러오고 표현을 하는 것이기 때문에 메모리 관리 면에서 더욱 더 예민할 수 밖에 없는 것 같다.
당장 나도 처음 할때만 해도 카메라를 켜놓고 있으면 혼자 미친 듯이 메모리를 잡아먹더니 1분도 안돼서 카메라가 죽어버리는 경우만 몇 번이었는지 기억도 안 난다.
