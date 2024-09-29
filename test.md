<p>以下<code>paddlex</code>命令中，新增了以下参数：</p>  
<pre><code>
paddlex \  
    --pipeline image_classification \  
    --input general_image_classification_001.jpg \  
    <span>--device gpu:0 \</span>  
    <span>--use_hpip \</span>  
    <span>--serial_number {序列号}</span>  
</code></pre>

<blockquote>
<div>
paddlex \  <br/>
    --pipeline image_classification \  <br/>
    --input general_image_classification_001.jpg \  <br/>
    <span>--device gpu:0 \</span>  <br/>
    <span>--use_hpip \</span>  <br/>
    <span>--serial_number {序列号}</span> 
</blockquote>

<blockquote>
<div>
paddlex \  <br/>
    --pipeline image_classification \  <br/>
    --input general_image_classification_001.jpg \  <br/>
    <span style="color: red;">--device gpu:0 \</span>  <br/>
    <span style="color: red;">--use_hpip \</span>  <br/>
    <span style="color: red;">--serial_number {序列号}</span> 
</blockquote>

<style>
span {color:red;}
</style>