<!--
<div class="page-content qrcode">

	thisURL  = document.URL;
	thisTitle= document.title;
 	<img src="http://qr.liantu.com/api.php?w=120&m=0&text=" + thisURL alt=thisTitle title="扫描二维码分享到朋友圈" />
 <p class="qrcode-desc">扫描二维码分享到朋友圈</p>
 <hr>
</div>
-->

<script type="text/javascript">  
  thisURL  = document.URL;  
  strwrite = "<p align='center'><img src='http://qr.liantu.com/api.php?w=120&m=0&text=" + thisURL + "' width='120' height='120' alt='QR Code'/>（传送门）</p>";
   document.write( strwrite ); 
</script>

<!--
<script type="text/javascript">  
  thisURL  = document.URL;  
  strwrite = "<p align='center'><img src='http://chart.apis.google.com/chart?chs=120x120&amp;cht=qr&amp;chld=|1&amp;chl=" + thisURL + "' width='120' height='120' alt='QR Code'/>（传送门）</p>";
   document.write( strwrite ); 
</script>
-->


