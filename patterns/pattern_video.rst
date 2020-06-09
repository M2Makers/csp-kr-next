.. _pattern-video:

동영상 서비스 패턴
******************

이 장에서는 동영상을 빠르고 호환성있게 전달하는 패턴에 대해 설명한다.
동영상 서비스에서는 :ref:`pattern-infra-2tier` 구조가 기본이 될 만큼 엄청난 저장공간과 전송대역폭을 필요로 한다.
막대한 비용이 지출되는 만큼 어떻게 안정적이고 경제적인 서비스를 구성할 것인가에 초점을 맞추어야 한다.



실시간 HLS 전송
====================================

해결하고 싶은 문제
------------------------------------
iOS를 기준으로 시장표준 동영상 전송방식은 HLS(Http Live Streaming)이다.
이미 스토리지에 MP4포맷으로 저장된 수 많은 영상을 MPEG-TS로 분할하기에는 시간과 공간의 제약이 크다.


솔루션/패턴 설명
------------------------------------
``STON`` 의 `MP4HLS <https://ston.readthedocs.io/ko/latest/admin/video.html#mp4-hls>`_ 기능을 이용해 실시간으로 MP4영상을 HLS로 가공하여 전송한다.

.. figure:: img/dgm011.png
   :align: center


구현
------------------------------------
-  동영상 스토리지/서비스 앞에 ``STON`` 을 배치한다.
-  ``STON`` MP4HLS 기능을 활성화한다. ::
   
      # vhosts.xml - <Vhosts><Vhost><Media>

      <MP4HLS Status="Active" Keyword="mp4hls">
         <Index Ver="3" Alternates="off">index.m3u8</Index>
         <Sequence>0</Sequence>
         <Duration>10</Duration>
         <AlternatesName>playlist.m3u8</AlternatesName>
      </MP4HLS>

-  플레이어에서 기존 동영상 주소 뒤에 ``~/mp4hls/index.m3u8`` 를 붙여 호출한다. ::

      http://www.example.com/video.mp4/mp4hls/index.m3u8



장점/효과
------------------------------------
별도의 백엔드 프로세스나 추가 저장공간의 소비 없이 즉시 HLS 서비스가 가능하다.


주의점
------------------------------------
``STON`` 은 MP4에 대한 트랜스코딩(transcoding)을 수행하지 않는다. 
따라서 원본 동영상의 Video/Audio를 다른 Bitrate로 재가공할 수 없다.


기타
------------------------------------
사용자가 동영상 전체를 시청하는 빈도가 낮다면 ``STON``  - `Range요청 <https://ston.readthedocs.io/ko/latest/admin/origin.html#range>`_ 기능을 이용해 스토리지/원본서버의 대역폭 소비를 최소화한다.



실시간 구간편집
====================================

해결하고 싶은 문제
------------------------------------
고객은 영상 전체보다 이슈 장면만 선별적으로 소비한다.
복잡한 사전작업 없이 이슈 즉시 편집된 영상을 제공하고 싶다.


솔루션/패턴 설명
------------------------------------
``STON`` 의 `Multi-Trimming <https://ston.readthedocs.io/ko/latest/admin/video.html#multi-trimming>`_ 기능을 이용해 실시간으로 MP4영상을 편집한다.

.. figure:: img/dgm012.png
   :align: center


구현
------------------------------------
-  동영상 스토리지/서비스 앞에 ``STON`` 을 배치한다.
-  ``STON`` MP4Trimming 기능을 활성화한다. ::
   
      # server.xml - <Server><VHostDefault><Media>
      # vhosts.xml - <Vhosts><Vhost><Media>

      <MP4Trimming MultiParam="trimming" MaxRatio="50">ON</MP4Trimming>
      <M4ATrimming MultiParam="trimming">ON</M4ATrimming>


-  플레이어에서 기존 동영상 주소 뒤에 원하는 구간을 초단위로 명시한다. ::

      http://example.com/video.mp4?trimming=10-70


장점/효과
------------------------------------
별도의 백엔드 프로세스나 추가 저장공간의 소비 없이 즉시 편집된 동영상 서비스가 가능하다.


주의점
------------------------------------
`ngx_http_mp4_module <http://nginx.org/en/docs/http/ngx_http_mp4_module.html>`_ 이나 `Mod-H264-Streaming-Testing-Version2 <http://h264.code-shop.com/trac/wiki/Mod-H264-Streaming-Testing-Version2>`_ 같은 사용방식을 원한다면 ``STON`` - `Trimming <https://ston.readthedocs.io/ko/latest/admin/video.html#trimming>`_ 을 사용한다.


기타
------------------------------------
`실시간 HLS 전송`_ 과 같이 사용할 수 있다. ::

   http://example.com/video.mp4?trimming=17-20,30-40/mp4hls/index.m3u8



