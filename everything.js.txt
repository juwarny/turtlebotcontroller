var rosManager = (function(){
  const gripper_open = [0.01];
  const gripper_close = [0.004]; //origin -0.01  bottle is 0.004
  const open = "open";
  const close = "close";
  const goal_destination = {
    header: {
      frame_id: "map"
    },
    pose:{
      position: {
        x : 0.20, // 수정
        y : 1.520,  // 수정
        z : 0.0
      },
      orientation : {
        x : 0.0,
        y : 0.0,
        z : 1.0, // 수정
        w : 0.004 // 수정
      }
    }
  };

  //TODO: 이부분은 약간 수정 필요
  const goal_init = {
    header: {
      frame_id: "map"
    },
    pose:{
      position: {
        x : 0.2, // 수정
        y : 0.2,  // 수정
        z : 0.0
      },
      orientation : {
        x : 0.0,
        y : 0.0,
        z : 1.0, // 수정
        w : 0.004 // 수정
      }
    }
  };


//0.01: gripper 벌리기
//-0.01: gripper 오므리기
// const gripper_open = [0.01];
// const gripper_close = [-0.01];
// const open = "open";
// const close = "close";
// const goal_destination = {
//   header: {
//     frame_id: "map"
//   },
//   pose:{
//     position: {
//       x : 0.236, // 수정
//       y : 1.520,  // 수정
//       z : 0.0
//     },
//     orientation : {
//       x : 0.0,
//       y : 0.0,
//       z : 0.004, // 수정
//       w : 1.0 // 수정
//     }
//   }
// };
//
// //TODO: 이부분은 약간 수정 필요
// const goal_init = {
//   header: {
//     frame_id: "map"
//   },
//   pose:{
//     position: {
//       x : 0.0, // 수정
//       y : 0.0,  // 수정
//       z : 0.0
//     },
//     orientation : {
//       x : 0.0,
//       y : 0.0,
//       z : 0.004, // 수정
//       w : 1.0 // 수정
//     }
//   }
// };

//connet
var ros = new ROSLIB.Ros({
  url: 'ws://192.168.0.30:9090/'
});

ros.on('connection', function() {
  console.log('Connected to websocket server.');
});

ros.on('error', function(error) {
  console.log('Error connecting to websocket server: ', error);
});

ros.on('close', function() {
  console.log('Connection to websocket server closed.');
});




  // Publishing a Topic
  // ------------------

  var navi = new ROSLIB.Topic({
    ros : ros,
    name : '/om_with_tb3/move_base_simple/goal',
    messageType : 'geometry_msgs/PoseStamped'
    });

  var navistatus = new ROSLIB.Topic({
    ros : ros,
    name : '/om_with_tb3/move_base/status',
    messageType : 'actionlib_msgs/GoalStatusArray'
  });

  var navicancel = new ROSLIB.Topic({
    ros : ros,
    name : '/om_with_tb3/move_base/cancel',
    messageType : 'actionlib_msgs/GoalID'
  });

  var cmdVel = new ROSLIB.Topic({
    ros: ros,
    name: '/om_with_tb3/cmd_vel',
    messageType: 'geometry_msgs/Twist'
  });

  var compressedImg = new ROSLIB.Topic({
    ros: ros,
    name: '/om_with_tb3/raspicam_node/image/compressed',
    messageType: 'sensor_msgs/CompressedImage'
  });


  var gripper = new ROSLIB.Service({
    ros: ros,
    name: '/om_with_tb3/gripper',
    serviceType: 'open_manipulator_msgs/SetJointPositionRequest'
  });

  var setJointPosition = new ROSLIB.Service({
    ros: ros,
    name: '/arm/moveit/set_joint_position',
    serviceType: 'open_manipulator_msgs/SetJointPositionRequest'
  });

  var getJointPosition = new ROSLIB.Service({
    ros: ros,
    name: '/arm/moveit/get_joint_position',
    serviceType: 'open_manipulator_msgs/GetJointPositionRequest'
  });

  var kinematicsSetJointPosition = new ROSLIB.Service({
    ros: ros,
    name: '/arm/moveit/set_kinematics_pose',
    serviceType: 'open_manipulator_msgs/SetKinematicsPoseRequest'
  });

  var kinematicsGetJointPosition = new ROSLIB.Service({
    ros: ros,
    name: '/arm/moveit/get_kinematics_pose',
    serviceType: 'open_manipulator_msgs/GetKinematicsPoseRequest'
  });

  var gripper_pos = {
    joint_name: [],
    position: [],//여기만 바꾸면 됨
    max_accelerations_scaling_factor: 0.0,
    max_velocity_scaling_factor: 0.0
  };

  var joint_pos = {
    joint_name: ["joint1", "joint2", "joint3", "joint4"],
    position: [0.0, 0.0, 0.0, 0.0],//이 값을 바꿔야 함
    max_accelerations_scaling_factor: 1.0,
    max_velocity_scaling_factor: 1.0
  };

  var kinematics_pose = {
    pose: {
      position:{
        x: 0.0,
        y: 0.0,
        z: 0.0
      },
      orientation:{
        x: 0.0,
        y: 0.0,
        z: 0.0,
        w: 0.0
      }
    },
    max_accelerations_scaling_factor: 1.0,
    max_velocity_scaling_factor: 1.0,
    tolerance: 0.01
  };

  var twistObject = {
    linear: {
      x: 0.0,
      y: 0.0,
      z: 0.0
    },
    angular: {
      x: 0.0,
      y: 0.0,
      z: 0.0
    }
  };


  /*
  	//좌회전
  	 angular : {
        x : 0.0,
        y : 0.0,
        z : 0.0
      }
  */
  //  cmdVel.publish(twist);
  // Calling a service
  // -----------------

  // joint1: 0- 가운데, 양수 - 왼쪽, 음수 - 오른쪽
  // joint2: 양수 - 모터가 아래로, 음수 - 모터가 위로
  // joint3: 양수 - 모터가 아래로, 음수 - 모터가 위로
  // joint4: 양수 - 모터가 아래로, 음수 - 모터가 위로
  // position : [0.0, 0.12, -0.24, 0.37],              //전진
  //  position : [0.0, -0.65, 1.20, -0.54],             //일반
var publishForMoveBaseGoal = function(goal, navi){
  var poseStamped = new ROSLIB.Message(goal);
  navi.publish(poseStamped);
};

var publishForCancelGoal = function(goalid, navicancel){
  console.log("start 246");
  console.log(goalid);
  var goal = {
    id: goalid
  };
  var poseStamped = new ROSLIB.Message(goal);

  console.log(navicancel);
  console.log(navicancel.messageType);
  navicancel.publish(poseStamped);
};

  var publishForCmdVel= function(twistObject, cmdVel){
    var twist = new ROSLIB.Message(twistObject);
    cmdVel.publish(twist);
  };

  var getToTarget = function (getJointPosition, callback){

    var request = new ROSLIB.ServiceRequest({
      planning_group: "arm",
      end_effector_name:""
    });

    getJointPosition.callService(request, callback);
  };

  var getToTargetKinematics = function (kinematicsGetJointPosition, callback){

    var request = new ROSLIB.ServiceRequest({
      planning_group: "arm",
      end_effector_name:""
    });

    kinematicsGetJointPosition.callService(request, callback);
  };

  var moveToTargetKinematics = function (location, kinematics_pose, kinematicsSetJointPosition, callback){
    kinematics_pose.pose = location;

    var request = new ROSLIB.ServiceRequest({
      planning_group: "arm",
      kinematics_pose: kinematics_pose,
      path_time: 0.0
    });

    kinematicsSetJointPosition.callService(request, callback);
  };


  var moveToTarget = function(location, joint_pos, setJointPosition, callback){
    joint_pos.position = location;

    var request = new ROSLIB.ServiceRequest({
      planning_group: "arm",
      joint_position: joint_pos,
      path_time: 0.0

    });

    setJointPosition.callService(request, callback);
  };

var closeOrOpenGrip = function (command, gripper_pos, gripper, callback){
  if(command==open){
    gripper_pos.position = gripper_open;
  }else{
    gripper_pos.position = gripper_close;
  }

  var gripper_request = new ROSLIB.ServiceRequest({
    planning_group: "",
    joint_position: gripper_pos,
    path_time: 0.0
  });

  gripper.callService(gripper_request, callback);
};

var getImgStream = function(callback){
  compressedImg.subscribe(callback);
}

var getGoalStatus = function(callback){
  navistatus.subscribe(callback);
}

return {
  ros : ros,
  navicancel: navicancel,
  navistatus: navistatus,
  publishForCancelGoal : publishForCancelGoal,
  getGoalStatus: getGoalStatus,
  getToTarget : getToTarget,
  getJointPosition : getJointPosition,
  getImgStream : getImgStream,
  compressedImg : compressedImg,
  gripper_open : gripper_open,
  gripper_close : gripper_close,
  open: open,
  close : close,
  goal_destination : goal_destination,
  goal_init : goal_init,
  navi : navi,
  cmdVel : cmdVel,
  gripper: gripper,
  setJointPosition : setJointPosition,
  kinematicsSetJointPosition:kinematicsSetJointPosition,
  gripper_pos : gripper_pos,
  joint_pos:joint_pos,
  kinematics_pose: kinematics_pose,
  twistObject: twistObject,
  publishForMoveBaseGoal : publishForMoveBaseGoal,
  publishForCmdVel: publishForCmdVel,
  moveToTargetKinematics:moveToTargetKinematics,
  moveToTarget:moveToTarget,
  closeOrOpenGrip:closeOrOpenGrip,
  getToTargetKinematics: getToTargetKinematics,
  kinematicsGetJointPosition: kinematicsGetJointPosition
}

})();
