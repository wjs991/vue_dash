<template>
  <div class="row">
    <div class="col-sm-6">
    <div class="card">
      <div class="header">
        Test
      </div>
      <div class="content" id="dd" @keydown="keyEvent">
        
      </div>
    </div>
    </div>
  </div>
  
</template>
<script>
export default {
  mounted() {
    var car;
    let scene = new THREE.Scene();
    var maze_Array = [];
    var n = 5;
    var room_num = 5;
    var maze = [[]];
    var plane = [];
    var m = [[]];
    //enter
    let maze_enter_x = 0;
    let maze_enter_z = 0;
    //escape
    let maze_escape_x = n - 1;
    let maze_escape_z = n - 1;
    var dd = window.document.getElementById("dd");
    var divWidth = dd.offsetWidth;
    var divHeight= dd.offsetHeight;

    var map_Size = n*2+1;
    fillEmptyMap(maze, map_Size, map_Size);
    fillEmptyMap(m,map_Size,map_Size);
    var list =[];
    list = new Array();
    var block= function(x,y,visited){
      this.x=x,
      this.y=y,
      this.visited=visited
      this.info=function(){
        return "x:"+this.x+" y:"+this.y+ " visited:"+this.visited;
      }
      
    }

    maze_init();
    var camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );
    
    
    console.log("height", dd.offsetheight);
    let renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(divWidth - 40, 400);

    dd.appendChild(renderer.domElement);

    //maze 바닥면 빌드
    for (var x = 0; x < map_Size; x++) {
      for (var z = 0; z < map_Size; z++) {
        let geometry = new THREE.BoxGeometry(1, 1, 1);

        let material = new THREE.MeshLambertMaterial({
          color: 0xeeeee,
          vertexColors: THREE.FaceColors
        });
        let cube = new THREE.Mesh(geometry, material);
        cube.position.x = 1.05 * x;
        cube.position.z = 1.05 * z;

        plane.push(cube);
        scene.add(cube);
      }
    }

    //maze빌드
    for (var x = 0; x < map_Size; x++) {
      for (var z = 0; z < map_Size; z++) {
        if (m[x][z] == 1) {
          let geometry = new THREE.BoxGeometry(1, 1, 1);

          let material = new THREE.MeshLambertMaterial({
            color: 0xf44542,
            vertexColors: THREE.FaceColors
          });
          let cube = new THREE.Mesh(geometry, material);
          cube.position.x = 1.05 * x;
          cube.position.y = 1.05;
          cube.position.z = 1.05 * z;

          maze_Array.push(cube);
          scene.add(cube);
        } else {
         
        }
      }
    }

    camera.position.z = 75;
    addLight();
    render();
    //window.on( 'resize', onWindowResize);
    function onWindowResize() {

		renderer.setSize( window.innerWidth, window.innerHeight );

	  }
    function addLight() {
      var light = new THREE.PointLight(0x9e15ec, 1, 100, 2);
      light.position.set(-4, 1, -1);
      scene.add(light);

      var light = new THREE.PointLight(0x0b8ae2, 1, 100, 2);
      light.position.set(4, 10, 1);
      scene.add(light);

      // Ambient light
      scene.add(new THREE.AmbientLight(0xbfbfbf));
    }

    function render() {
      requestAnimationFrame(render);
      camera.lookAt(new THREE.Vector3(n / 2, 0, n / 2));
      camera.position.x = n*2;
      camera.position.y = n*2;
      camera.position.z = n*2;
      renderer.render(scene, camera);
    }

    function fillEmptyMap(array, width, height) {
      for (var x = 0; x < width+1; x++) {
        array[x] = [];
        for (var y = 0; y < height+1; y++) {
          array[x][y] = -1;
          //console.log(array[x][y]);
        }
      }
    }

    function maze_init(){
      var map_Size_up = map_Size+1
      for(var i = 0; i< map_Size_up-1;i++){
        for(var j= 0; j< map_Size_up-1;j++){
          maze[i][j] =new block(i,j,false); //여기서 왜 
        }
      }

      generateMap();
    }

    function generateMap(){
      var ranX = (Math.floor(Math.random()*n))*2+1;

      var ranY = (Math.floor(Math.random()*n))*2+1;

      maze[ranY][ranX].visited = true;
      
      if(ranX-1 !=0){
        list.push(maze[ranY][ranX-1]);
      }

      if(ranX+1!=map_Size){
        list.push(maze[ranY][ranX+1]);  //???
      }

      if(ranY-1 !=0){
        list.push(maze[ranY-1][ranX]);
      }
      
      if(ranY+1 !=map_Size){
        list.push(maze[ranY+1][ranX]);
      }

      console.log(list);
      while(list.length!=0){
        var index = Math.floor((Math.random()*list.length));
        
        var wall = list[index];
        //x가 범위를 넘어가네? 수정이 필요하다!  메이즈 초기화하는 부분에서 생기는 오류인거같다.

        if(wall.y %2 ==1){

            if((wall.x)-1!=0&&maze[wall.y][(wall.x)-1].visited === false){
              maze[wall.y][wall.x-1].visited = true;
              maze[wall.y][wall.x].visited = true;

              if((wall.x)-2!=0){
                list.push(maze[wall.y][(wall.x)-2]);
              }
              if((wall.y)-1!=0){
                list.push(maze[(wall.y)-1][(wall.x)-1]);
              }
              if((wall.y)+1!=0){
                list.push(maze[(wall.y)+1][(wall.x)-1]);
              }

            }else if((wall.x)+1!=0&&maze[wall.y][(wall.x+1)].visited === false){
              maze[wall.y][(wall.x)+1].visited = true;
              maze[wall.y][wall.x].visited = true;

               if(wall.x+2!=0){
                list.push(maze[wall.y][(wall.x)+2]);
              }
              if(wall.y-1!=0){
                list.push(maze[(wall.y)-1][(wall.x)+1]);
              }
              if(wall.y+1!=0){
                list.push(maze[(wall.y)+1][(wall.x)+1]);
              }
            }
        }else{
          if((wall.y)-1 !=0 && maze[(wall.y)-1][wall.x].visited ===false){
              maze[(wall.y)-1][wall.x].visited =true;
              maze[wall.y][wall.x].visited = true;

            if((wall.y)-2!=0){
                list.push(maze[(wall.y)-2][wall.x]);
              }
              if((wall.x)-1!=0){
                list.push(maze[(wall.y)-1][(wall.x)-1]);
              }
              if((wall.x)+1!=0){
                list.push(maze[(wall.y)-1][(wall.x)+1]);
              }
          
          
         }else if((wall.y)+1 <= map_Size-1 && maze[wall.y+1][wall.x].visited ===false){
              maze[wall.y+1][wall.x].visited = true;
              maze[wall.y][wall.x].visited = true;

              if(wall.y+2!=0){
                list.push(maze[wall.y+2][wall.x]);
              }
              if(wall.x-1!=0){
                list.push(maze[wall.y+1][wall.x-1]);
              }
              if(wall.x+1!=0){
                list.push(maze[wall.y+1][wall.x+1]);
              }
          }
        }
        console.log("삭제전");
        console.log(list);
        var a=list.splice(index,1);
        console.log("삭제후");
        console.log(a);
      }

      for(var i = 0 ;i<map_Size;i++){
        for(var j= 0 ; j<map_Size ;j++){
          if(maze[i][j].visited === false){
            if(i%2 ===1){
              if(j%2 ===0){
                m[i][j]=1;
              }
            }else{
              m[i][j]=1;
            }
          }
        }
      }

      for(var i=0;j<map_Size;j++){
        m[i][map_Size-1]=1;
        m[i][0]=1;

        m[map_Size-1][i]=1;
        m[0][i] = 1;
      }

      m[0][1]=0;
      m[map_Size-1][map_Size-2] = 0;


    }
    window.addEventListener('keydown',this.keyEvent,true);

    
  },
  methods:{
    keyEvent: function(e){
      var key ='';
      switch(e.key) {
      	case 'ArrowUp':
        	key = 'top'
          console.log(e.key);
        	break
        case 'ArrowDown':
        	key = 'top'
	        break
        case 'ArrowLeft':
        	key = 'left'
          
        	break
        case 'ArrowRight':
        	key = 'left'
        	break
        default:
        	return
      }
    }
  }
};
</script>
<style>

</style>
