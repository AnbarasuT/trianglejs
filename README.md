window.Triangle=function(canvas,Options){
  this.canvas=document.getElementById(canvas);
  if (!Options) Options = {};
  this.Options = Options;
  this.fillStyle=this.Options.fillStyle? this.Options.fillStyle:"transparent";
  this.lineWidth=this.Options.lineWidth? this.Options.lineWidth:2;
  this.context= this.canvas.getContext("2d");
  this.controlPoints=[];
  this.controlPointRadius = 6;
  this.touchpointradius = 15;
  this.triPoints = this.Options.triPoints ? this.Options.triPoints:[10,100,810,250,550,286];
  this.triangle = new this.tris(this.triPoints);
  this.pointDistance = this.Options.pointDistance ? this.Options.pointDistance : {};
  this.selectedIndex=-1;
  this.isDrag = false;
  this.commonValue = 0.028;
  this.init();
};
window.Triangle.prototype={
  init:function(){
    var _this = this;
    $(this.canvas).on('mousedown touchstart',function(event){ _this.touchStart(event)});
    $(window).on('mousemove touchmove',function(event){ _this.touchMove(event)});
    $(window).on('mouseup touchend',function(event){ _this.touchEnd(event)});
    this.controlPoints[0]=this.point(this.triangle.x1,this.triangle.y1);
    this.controlPoints[1]=this.point(this.triangle.x2,this.triangle.y2);
    this.controlPoints[2]=this.point(this.triangle.x3,this.triangle.y3);
    this.updateControlPoints();
    this.drawAll();
  },
  loadTriangle:function(points){
    this.triangle = new this.tris(points);
    this.controlPoints[0]=this.point(this.triangle.x1,this.triangle.y1);
    this.controlPoints[1]=this.point(this.triangle.x2,this.triangle.y2);
    this.controlPoints[2]=this.point(this.triangle.x3,this.triangle.y3);
    this.updateControlPoints();
    this.drawAll();
  },
  drawAll:function(){
    this.context.clearRect(0,0,this.canvas.width,this.canvas.height);
    if (this.canvas.id == "canvas_main1"){
      var midPoint = this.pointDistance.midPoint;
      this.context.beginPath();
      this.context.arc(midPoint.x,midPoint.y,distance,0,2*Math.PI);
      this.context.lineWidth = this.lineWidth;
      this.context.strokeStyle="#08644D";
      this.context.closePath();
      this.context.stroke();
      this.context.beginPath();
      this.context.arc(midPoint.x,midPoint.y,distance1,0,2*Math.PI);
      this.context.strokeStyle="#08644D";
      this.context.lineWidth = this.lineWidth;
      this.context.closePath();
      this.context.stroke();
    }
    if (this.canvas.id == "canvas_main"){
      this.context.font='bold 18px Gill Sans Infant MT';
      this.context.fillStyle="#000";
      this.context.fillText("A", this.triangle.x1-10,this.triangle.y1+30);
      this.context.fillText("B", this.triangle.x2-5,this.triangle.y2-15);
      this.context.fillText("C", this.triangle.x3-5,this.triangle.y3+30);
    }
    if (this.canvas.id == "canvas_main1"){
      this.context.font='bold 20px Gill Sans Infant MT';
      this.context.fillStyle="#000";
      this.context.fillText("F", this.triangle.x1+5,this.triangle.y1+30);
      this.context.fillText("D", this.triangle.x2-10,this.triangle.y2-15);
      this.context.fillText("E", this.triangle.x3+20,this.triangle.y3+20);
    }
    this.context.beginPath();
    this.context.moveTo(this.triangle.x1,this.triangle.y1);
    this.context.lineTo(this.triangle.x2,this.triangle.y2);
    this.context.lineTo(this.triangle.x3,this.triangle.y3);
    var m1=(this.triangle.y2-this.triangle.y1)/(this.triangle.x2-this.triangle.x1);
    var m2=(this.triangle.y3-this.triangle.y2)/(this.triangle.x3-this.triangle.x2);
    //this.context.fill();
    this.context.strokeStyle = '#0442BF';
    this.context.lineWidth = this.lineWidth;
    this.context.closePath();
    this.context.stroke();
    
    this.context.save();
    for(var i=0;i<this.controlPoints.length;i++){
      this.context.fillStyle="#E34242"; 
      this.context.beginPath();
      this.context.arc(this.controlPoints[i].x,this.controlPoints[i].y,this.controlPointRadius,0,2*Math.PI);
      this.context.fill();
      this.context.strokeStyle = '#000';
      this.context.lineWidth = this.lineWidth;
      this.context.stroke();
      this.context.closePath();
    }
    this.context.lineWidth = this.lineWidth;
    this.context.restore();
  },
  tris:function(tripoints){
    this.x1=tripoints[0];
    this.y1=tripoints[1];
    this.x2=tripoints[2];
    this.y2=tripoints[3];
    this.x3=tripoints[4];
    this.y3=tripoints[5];
  },
  updateControlPoints:function(){
    this.triangle.x1=this.controlPoints[0].x;
    this.triangle.y1=this.controlPoints[0].y;
    this.triangle.x2=this.controlPoints[1].x;
    this.triangle.y2=this.controlPoints[1].y;
    this.triangle.x3=this.controlPoints[2].x;
    this.triangle.y3=this.controlPoints[2].y;
  },
  touchStart:function(event){
    
        try {
	  tx = event.originalEvent.touches[0].pageX-$(this.canvas).offset().left;
	  ty = event.originalEvent.touches[0].pageY-$(this.canvas).offset().top;
        } catch (e) {
	  tx=event.pageX-$(this.canvas).offset().left;
	  ty=event.pageY-$(this.canvas).offset().top;
        }
    //tx=event.pageX-$(this.canvas).offset().left;
    //ty=event.pageY-$(this.canvas).offset().top;
    for (var cindex=0;cindex<this.controlPoints.length;cindex++) {
      cx=tx-this.controlPoints[cindex].x;
      cy=ty-this.controlPoints[cindex].y;
      console.log("CCCC"+cx,cy);
      if ((cx*cx)+(cy*cy)<(this.touchpointradius*this.touchpointradius) ){
	this.isDrag=true;
	this.selectedIndex=cindex;
      console.log("////touch point: "+this.selectedIndex,cx,cy);

	break;
      }
    }
  },
  touchMove:function(event){
    if (this.isDrag) {
      this.triggerChange();
      if (this.canvas.id == "canvas_main")
      {
      var cmy = Math.sqrt(Math.pow(this.triangle.y1-this.triangle.y3,2) + Math.pow(this.triangle.x1-this.triangle.x3,2))*this.commonValue;
      $('#cmyvalue').find('input').val(cmy.toFixed(2)+" "+"cm");
      
      var cmx = Math.sqrt(Math.pow(this.triangle.y1-this.triangle.y2,2) + Math.pow(this.triangle.x1-this.triangle.x2,2))*this.commonValue;
      $('#cmxvalue').find('input').val(cmx.toFixed(2)+" "+"cm");
      
      var cmz = Math.sqrt(Math.pow(this.triangle.y3-this.triangle.y2,2) + Math.pow(this.triangle.x3-this.triangle.x2,2))*this.commonValue;
      $('#cmzvalue').find('input').val(cmz.toFixed(2)+" "+"cm");
      }
      //var cmy1 = (Math.max(this.triangle.x1,this.triangle.y1)-(Math.min(this.triangle.x1,this.triangle.y1)))*0.026458333;
      //$('#cmy1value').find('input').val(cmy1.toFixed(2)+" "+"cm");
      
	//var cmx1 =(Math.max(this.triangle.x2,this.triangle.y2)-(Math.min(this.triangle.x2,this.triangle.y2)))*0.026458333;
	//$('#cmx1value').find('input').val(cmx1.toFixed(2)+" "+"cm");
	//     
	//F x1,y1//
      if (this.canvas.id == "canvas_main1" && this.selectedIndex == 0){
	var cmy1 = (Math.max(this.triangle.x1,this.triangle.y1)-(Math.min(this.triangle.x1,this.triangle.y1)))*0.026458333;
	$('#cmy1value').find('input').val(cmy1.toFixed(2)+" "+"cm");
	console.log("Sel F: ")

      }
      //D x2,y2//
      if (this.canvas.id == "canvas_main1" && this.selectedIndex == 1){
      var cmx1 = Math.sqrt(Math.pow(this.triangle.y1-this.triangle.y2,2) + Math.pow(this.triangle.x1-this.triangle.x2,2))*0.028978333;
      $('#cmx1value').find('input').val(cmx1.toFixed(2)+" "+"cm");
	console.log(Math.sqrt(Math.pow(this.triangle.y1-this.triangle.y2,2) + Math.pow(this.triangle.x1-this.triangle.x2,2)));
	console.log("Sel D: "+this.triangle.y1,this.triangle.x1);

      var cmz1 = Math.sqrt(Math.pow(this.triangle.y3-this.triangle.y2,2) + Math.pow(this.triangle.x3-this.triangle.x2,2))*0.026458333;
	$('#cmz1value').find('input').val(cmz1.toFixed(2)+" "+"cm");	
      }
      
      //E//
      if (this.canvas.id == "canvas_main1" && this.selectedIndex == 2){
        var cmy1 = Math.sqrt(Math.pow((this.triangle.y1-this.triangle.y3), 2) + Math.pow((this.triangle.x1-this.triangle.x3), 2))*0.033658333;

        //var cmy1 = Math.sqrt(Math.pow(this.triangle.y1-this.triangle.y3,2) + Math.pow(this.triangle.x1-this.triangle.x3,2))*0.026458333;
	$('#cmy1value').find('input').val(cmy1.toFixed(2)+" "+"cm");
	console.log("EF"+Math.sqrt(Math.pow((this.triangle.y1-this.triangle.y3), 2) + Math.pow((this.triangle.x1-this.triangle.x3), 2)));
	
        var cmz1 = Math.sqrt(Math.pow(this.triangle.y3-this.triangle.y2,2) + Math.pow(this.triangle.x3-this.triangle.x2,2))*0.026458333;
	
	//var cmz1 =(Math.max(this.triangle.x2,this.triangle.y3)-(Math.min(this.triangle.x2,this.triangle.y3)))*0.026458333;
	$('#cmz1value').find('input').val(cmz1.toFixed(2)+" "+"cm");	
	
	console.log("Sel E: ")
      }
      
      
      /*if (this.canvas.id == "canvas_main1" && this.selectedIndex == 2){
	var cmz1 = (Math.max(this.triangle.x3,this.triangle.y3)-(Math.min(this.triangle.x3,this.triangle.y3)))*0.026458333;
	$('#cmz1value').find('input').val(cmz1.toFixed(2)+" "+"cm");
      }
      
      
      if (this.canvas.id == "canvas_main1" && this.selectedIndex == 1){
	var cmx1 =(Math.max(this.triangle.x2,this.triangle.y2)-(Math.min(this.triangle.x2,this.triangle.y2)))*0.026458333;
	$('#cmx1value').find('input').val(cmx1.toFixed(2)+" "+"cm");
      }
      if (this.canvas.id == "canvas_main" && this.selectedIndex == 2){
	var cmz1 = (Math.max(this.triangle.x3,this.triangle.y3)-(Math.min(this.triangle.x3,this.triangle.y3)))*0.026458333;
	$('#cmz1value').find('input').val(cmz1.toFixed(2)+" "+"cm");
      }*/
      
     // console.clear();
      var angle_AC = Math.atan2(this.triangle.y3-this.triangle.y1, this.triangle.x3-this.triangle.x1)*180/Math.PI;
      var angle_AB = Math.atan2(this.triangle.y2-this.triangle.y1, this.triangle.x2-this.triangle.x1)*180/Math.PI;
      var angle_BC = Math.atan2(this.triangle.y3-this.triangle.y2, this.triangle.x3-this.triangle.x2)*180/Math.PI;
      if (angle_AC <= 0) angle_AC += 360;
      if (angle_AB <= 0) angle_AB += 360;
      if (angle_BC <= 0) angle_BC += 360;
      
      var angle_A = angle_AC-angle_AB;
      var angle_C = angle_BC-angle_AC;
      
      if (angle_A <= 0) angle_A += 360;
      if (angle_C <= 0) angle_C += 360;
      
      if (angle_A > 180) angle_A = 360-angle_A;
      if (angle_C > 180) angle_C = 360-angle_C;
      
      var angle_B = 180-(angle_A+angle_C);
      
      angle_A = Math.round(angle_A);
      angle_B = Math.round(angle_B);
      angle_C = Math.round(angle_C);
      if (this.canvas.id == "canvas_main"){
	  $('#angleA').find('input').val(angle_A.toFixed(2)+""+String.fromCharCode(176));
	  $('#angleB').find('input').val(angle_B.toFixed(2)+""+String.fromCharCode(176));
	  $('#angleC').find('input').val(angle_C.toFixed(2)+""+String.fromCharCode(176));
      }
      //console.log(angle_A, angle_B, angle_C);
        try {
	  tx = event.originalEvent.touches[0].pageX-$(this.canvas).offset().left;
	  ty = event.originalEvent.touches[0].pageY-$(this.canvas).offset().top;
        } catch (e) {
	  tx=event.pageX-$(this.canvas).offset().left;
	  ty=event.pageY-$(this.canvas).offset().top;
        }
	

      //tx=event.pageX-$(this.canvas).offset().left;
      //ty=event.pageY-$(this.canvas).offset().top;
	tx<30?tx=30:'';
	tx>460 ? tx=460:'';
	ty<50 ? ty=50:'';
	ty>470?ty=470:'';
	if (tx>=30 && tx<=460 && ty>=50 && ty<=470) {
	  this.context.clearRect(0,0,this.canvas.width,this.canvas.height);
	  if(this.pointDistance.distance){
	    var midPoint = this.pointDistance.midPoint;
	    var pointDist = this.pointDistance.distance[this.selectedIndex];
	    if (pointDist > 0) {
	      var dx = midPoint.x - tx;
	      var dy = midPoint.y - ty;
	      var angle = Math.atan2(dy, dx)+Math.PI;
	      console.log("***"+tx,ty,dx,dy);
	      //
	      //var r1x = (tx * Math.cos(angle));
	      //var r1y = (ty * Math.sin(angle));
	      //var mpx = tx-250;
	      //var mpy = ty-260;
	      //mpx=mpx-tx;
	      //mpy=mpy-ty;
	      if (pointDist==distance)
	      {
	      //distance = tx-250;
	      console.log(this.pointDistance.distance[2]);
	      console.log("==="+distance,tx,ty);
	      tx = midPoint.x + (distance) * Math.cos(angle);
	      ty = midPoint.y + (distance) * Math.sin(angle);
	      }
	      if (pointDist==distance1)
	      {
	      tx = midPoint.x + (pointDist) * Math.cos(angle);
	      ty = midPoint.y + (pointDist) * Math.sin(angle);
	      }
	  //console.log(tx,ty,pointDist,dx,dy,r1x,r1y,mpx,mpy);
	    }
	    else{
	      tx = midPoint.x;
	      ty = midPoint.y;
	    }
	  }
	  this.controlPoints[this.selectedIndex].x=tx;
	  this.controlPoints[this.selectedIndex].y=ty;
	  this.updateControlPoints();
	  this.OnTouchMoveTrigger();
      }
    }
  },
  touchEnd:function(event){
    this.isDrag=false;
  },
  triggerChange:function(){
    
  },
  point:function(xPoint, yPoint){
    return {x:xPoint, y:yPoint}
  },
  circle:function(xPoint, yPoint, radius){
    return {x:xPoint, y:yPoint, radius:(radius?radius:this.controlPointRadius)}
  },
  OnTouchMoveTrigger:function(){}
}
