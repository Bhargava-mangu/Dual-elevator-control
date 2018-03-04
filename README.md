
`timescale 1ps/1ps // time scale of 1ns and resolution 0.01 ns

module de_ct ( input clk, reset, req_FF, req_SF, 
                              req_E1_F2S, req_E1_S2F, 
                              req_E2_F2S, req_E2_S2F, 
                              obs_sensor1, obs_sensor2,
                              open_timer1, open_timer2,
                                 
			output reg [2:0]E1_pos, E2_pos );
			reg [2:0]E1_next_pos, E2_next_pos;
			reg [25:0]a;
    
/*    Details of parameter for the postion sensors : E1_pos && E2_pos (and next)
                     Parameter      ||  Code  ||    Elevator_position
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~*/
   parameter firstfloor_closedoor = 3'b000,  //  Elevator @ First Floor with closed door.
        firstfloor_opendoor = 3'b001,  //  Elevator @ First Floor with open door.
        moving_first2second = 3'b010,  //  Elevator Moving from First to Second Floor.
        secondfloor_closeddoor = 3'b011,  //  Elevator @ Second Floor with closed door.
        secondfloor_opendoor = 3'b100,  //  Elevator @ Second Floor with open door.
        moving_second2first = 3'b101;  //  Elevator Moving from Second to First Floor.
   
   // Block to reset or assign the previous next state to the present state


	always@(posedge clk) begin // normal clock used for simulation
      if (reset)               // On reset the elevators moves to its respective priority floor.
         begin
         E1_pos <= firstfloor_closedoor;     // Elevator1 has priority for first floor.
         E2_pos <= secondfloor_closeddoor;     // Elevator2 has priority for second floor.
         end
      else
         begin
         E1_pos <= E1_next_pos;  
         E2_pos <= E2_next_pos;
         end
      end
   
   // Block to define the next state for each present state w.r.t. the various inputs signals.
   always@(E1_pos, E2_pos, req_FF, req_SF, req_E1_F2S, req_E1_S2F, req_E2_F2S, req_E2_S2F, obs_sensor1, obs_sensor2, open_timer1, open_timer2 ) begin

      case (E1_pos)
          firstfloor_closedoor : begin
                case (E2_pos)
                     firstfloor_closedoor : begin
                            if(req_FF || req_E1_S2F) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = firstfloor_closedoor;  end
                            else if(req_SF || req_E2_F2S) begin
                              E1_next_pos = firstfloor_closedoor;  
                              E2_next_pos = moving_first2second;  end
                            else if(req_E1_F2S) begin
                              E1_next_pos = moving_first2second;  
                              E2_next_pos = firstfloor_closedoor;  end
                            else if(req_E2_S2F) begin
                              E1_next_pos = firstfloor_closedoor;  
                              E2_next_pos = firstfloor_opendoor;  end  // 1
                            else begin
                              E1_next_pos = E1_pos;  
                              E2_next_pos = E2_pos;  end // 1.1
                            end
                            
                     firstfloor_opendoor : begin
                           if(req_FF || req_E2_S2F) begin
                             E1_next_pos = firstfloor_closedoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(req_SF || req_E1_F2S) begin
                             E1_next_pos = moving_first2second;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(req_E1_S2F) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if (!obs_sensor2 && open_timer2 && req_E2_F2S) begin
                             E1_next_pos = firstfloor_closedoor;  
                             E2_next_pos = moving_first2second;  end     
                           else if (!obs_sensor2 && open_timer2 ) begin
                             E1_next_pos = firstfloor_closedoor;  
                             E2_next_pos = firstfloor_closedoor;  end  // 2   
                            else begin
                              E1_next_pos = E1_pos;  
                              E2_next_pos = E2_pos;  end // 2.1
                           end
                           
                     moving_first2second  : begin
                            if(req_FF || req_E1_S2F) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = secondfloor_opendoor;  end
                            else if(req_SF || req_E2_F2S || req_E2_S2F) begin
                              E1_next_pos = firstfloor_closedoor;  
                              E2_next_pos = secondfloor_opendoor;  end
                            else if(req_E1_F2S) begin
                              E1_next_pos = moving_first2second;  
                              E2_next_pos = secondfloor_opendoor;  end // 3
                            else begin
                              E1_next_pos = E1_pos;  
                              E2_next_pos = secondfloor_opendoor;  end // 3.1
                            end
                            
                     secondfloor_closeddoor : begin
                              if(req_FF || req_E1_S2F) begin
                                E1_next_pos = firstfloor_opendoor;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(req_SF || req_E2_F2S) begin
                                E1_next_pos = firstfloor_closedoor;  
                                E2_next_pos = secondfloor_opendoor;  end
                              else if(req_E1_F2S) begin
                                E1_next_pos = moving_first2second;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(req_E2_S2F) begin
                                E1_next_pos = firstfloor_closedoor;  
                                E2_next_pos = moving_second2first;  end // 4
                             else begin
                               E1_next_pos = E1_pos;  
                               E2_next_pos = E2_pos;  end // 4.1
                              end
                              
                     secondfloor_opendoor  : begin
                             if(req_FF || req_E1_S2F) begin
                               E1_next_pos = firstfloor_opendoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(req_SF || req_E2_F2S) begin
                               E1_next_pos = firstfloor_closedoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(req_E1_F2S) begin
                               E1_next_pos = moving_first2second;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(!obs_sensor2 && open_timer2 && req_E2_S2F) begin
                               E1_next_pos = firstfloor_closedoor;  
                               E2_next_pos = moving_second2first;  end 
                             else if(!obs_sensor2 && open_timer2 ) begin
                               E1_next_pos = firstfloor_closedoor;  
                               E2_next_pos = secondfloor_closeddoor;  end // 5
                             else begin
                               E1_next_pos = E1_pos;  
                               E2_next_pos = E2_pos;  end // 5.1
                             end
                             
                     moving_second2first : begin
                           if(req_FF || req_E1_S2F) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(req_SF || req_E1_F2S) begin
                             E1_next_pos = moving_first2second;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(req_E2_F2S || req_E2_S2F) begin
                             E1_next_pos = firstfloor_closedoor;  
                             E2_next_pos = firstfloor_opendoor;  end // 6
                           else begin
                             E1_next_pos = E1_pos;  
                             E2_next_pos = firstfloor_opendoor;  end // 6.1
                           end
                           
                     default : begin
                         E1_next_pos = E1_pos;
                         E2_next_pos = E2_pos;
                         end
                 endcase
            end
            
          firstfloor_opendoor : begin
                case (E2_pos)
                     firstfloor_closedoor : begin
                            if(req_FF || req_E1_S2F) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = firstfloor_closedoor;  end
                            else if(req_SF || req_E2_F2S) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = moving_first2second;  end
                            else if(!obs_sensor1 && open_timer1 && req_E1_F2S) begin
                              E1_next_pos = moving_first2second;  
                              E2_next_pos = firstfloor_closedoor;  end
                            else if(req_E2_S2F) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = firstfloor_opendoor;  end 
                            else if(!obs_sensor1 && open_timer1 ) begin
                              E1_next_pos = firstfloor_closedoor;  
                              E2_next_pos = firstfloor_closedoor;  end // 7
                            else begin
                              E1_next_pos = E1_pos;  
                              E2_next_pos = E2_pos;  end // 7.1
                            end
                            
                     firstfloor_opendoor : begin
                           if(req_FF || req_E1_S2F || req_E2_S2F) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if((!obs_sensor2 && open_timer2) && (req_SF || req_E2_F2S)) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = moving_first2second;  end
                           else if(!obs_sensor1 && open_timer1 && req_E1_F2S) begin
                             E1_next_pos = moving_first2second;  
                             E2_next_pos = firstfloor_opendoor;  end 
                           else if((!obs_sensor2 && open_timer2)) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = firstfloor_closedoor;  end
                           else if(!obs_sensor1 && open_timer1 ) begin
                             E1_next_pos = firstfloor_closedoor;  
                             E2_next_pos = firstfloor_opendoor;  end // 8
                           else begin
                             E1_next_pos = E1_pos;  
                             E2_next_pos = E2_pos;  end // 8.1
                           end
                           
                     moving_first2second  : begin
                            if(req_FF || req_SF || req_E1_S2F || req_E2_F2S || req_E2_S2F) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = secondfloor_opendoor;  end
                            else if(!obs_sensor1 && open_timer1 && req_E1_F2S) begin
                              E1_next_pos = moving_first2second;  
                              E2_next_pos = secondfloor_opendoor;  end 
                            else if(!obs_sensor1 && open_timer1 ) begin
                              E1_next_pos = firstfloor_closedoor;  
                              E2_next_pos = secondfloor_opendoor;  end // 9
                            else begin
                              E1_next_pos = E1_pos;  
                              E2_next_pos = secondfloor_opendoor;  end // 9.1
                            end
                            
                     secondfloor_closeddoor : begin
                              if(req_FF || req_E1_S2F) begin
                                E1_next_pos = firstfloor_opendoor;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(req_SF || req_E2_F2S) begin
                                E1_next_pos = firstfloor_opendoor;  
                                E2_next_pos = secondfloor_opendoor;  end
                              else if(!obs_sensor1 && open_timer1 && req_E1_F2S) begin
                                E1_next_pos = moving_first2second;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(req_E2_S2F) begin
                                E1_next_pos = firstfloor_opendoor;  
                                E2_next_pos = moving_second2first;  end 
                              else if(!obs_sensor1 && open_timer1) begin
                                E1_next_pos = firstfloor_closedoor;  
                                E2_next_pos = secondfloor_closeddoor;  end // 10
                              else begin
                                E1_next_pos = E1_pos;  
                                E2_next_pos = E2_pos;  end // 10.1
                              end
                              
                     secondfloor_opendoor  : begin
                             if(req_FF || req_SF || req_E1_S2F || req_E2_F2S) begin
                               E1_next_pos = firstfloor_opendoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(!obs_sensor1 && open_timer1 && req_E1_F2S) begin
                               E1_next_pos = moving_first2second;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(!obs_sensor2 && open_timer2 && req_E2_S2F) begin
                               E1_next_pos = firstfloor_opendoor;  
                               E2_next_pos = moving_second2first;  end 
                             else if(!obs_sensor1 && open_timer1 ) begin
                               E1_next_pos = firstfloor_closedoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(!obs_sensor2 && open_timer2 ) begin
                               E1_next_pos = firstfloor_opendoor;  
                               E2_next_pos = secondfloor_closeddoor;  end // 11
                             else begin
                               E1_next_pos = E1_pos;  
                               E2_next_pos = E2_pos;  end // 11.1
                             end
                             
                     moving_second2first : begin
                           if(req_FF || req_E1_S2F || req_E2_F2S || req_E2_S2F) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if((!obs_sensor1 && open_timer1) && (req_SF || req_E1_F2S)) begin
                             E1_next_pos = moving_first2second;  
                             E2_next_pos = firstfloor_opendoor;  end 
                           else if(!obs_sensor1 && open_timer1) begin
                             E1_next_pos = firstfloor_closedoor;  
                             E2_next_pos = firstfloor_opendoor;  end // 12
                           else begin
                             E1_next_pos = E1_pos;  
                             E2_next_pos = firstfloor_opendoor;  end // 12.1
                           end
                           
                     default : begin
                         E1_next_pos = E1_pos;
                         E2_next_pos = E2_pos;
                         end
                 endcase
            end
            
          moving_first2second  : begin
                case (E2_pos)
                     firstfloor_closedoor : begin
                            if(req_FF || req_E2_S2F) begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = firstfloor_opendoor;  end
                            else if(req_SF || req_E1_F2S || req_E1_S2F) begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = firstfloor_closedoor;  end
                            else if(req_E2_F2S) begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = moving_first2second;  end // 13
                            else begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = E2_pos;  end // 13.1
                            end
                            
                     firstfloor_opendoor : begin
                           if(req_FF || req_SF || req_E1_F2S || req_E1_S2F || req_E2_S2F) begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(!obs_sensor2 && open_timer2 && req_E2_F2S) begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = moving_first2second;  end 
                           else if(!obs_sensor2 && open_timer2 ) begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = firstfloor_closedoor;  end // 14
                           else begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = E2_pos;  end // 14.1
                           end
                           
                     moving_first2second  : begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = secondfloor_opendoor;  /// 15
                            end
                            
                     secondfloor_closeddoor : begin
                              if(req_FF || req_E2_S2F) begin
                                E1_next_pos = secondfloor_opendoor;  
                                E2_next_pos = moving_second2first;  end
                              else if(req_SF || req_E2_F2S) begin
                                E1_next_pos = secondfloor_opendoor;  
                                E2_next_pos = secondfloor_opendoor;  end
                              else if(req_E1_F2S || req_E1_S2F) begin
                                E1_next_pos = secondfloor_opendoor;  
                                E2_next_pos = secondfloor_closeddoor;  end // 16
                              else begin
                                E1_next_pos = secondfloor_opendoor;  
                                E2_next_pos = E2_pos;  end // 16.1
                              end
                              
                     secondfloor_opendoor  : begin
                             if((!obs_sensor2 && open_timer2) && (req_FF || req_E2_S2F)) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = moving_second2first;  end
                             else if(req_SF || req_E1_F2S || req_E1_S2F || req_E2_F2S) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = secondfloor_opendoor;  end 
                             else if(!obs_sensor2 && open_timer2) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = secondfloor_closeddoor;  end // 17
                             else begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = E2_pos;  end // 17.1
                             end
                             
                     moving_second2first : begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  /// 18
                           end
                           
                     default : begin
                         E1_next_pos = secondfloor_opendoor;
                         E2_next_pos = E2_pos;
                         end
                 endcase
            end
            
          secondfloor_closeddoor : begin
                case (E2_pos)
                     firstfloor_closedoor : begin
                            if(req_FF || req_E2_S2F) begin
                              E1_next_pos = secondfloor_closeddoor;  
                              E2_next_pos = firstfloor_opendoor;  end
                            else if(req_SF || req_E1_F2S) begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = firstfloor_closedoor;  end
                            else if(req_E1_S2F) begin
                              E1_next_pos = moving_second2first;  
                              E2_next_pos = firstfloor_closedoor;  end
                            else if(req_E2_F2S) begin
                              E1_next_pos = secondfloor_closeddoor;  
                              E2_next_pos = moving_first2second;  end // 19
                            else begin
                              E1_next_pos = E1_pos;  
                              E2_next_pos = E2_pos;  end // 19.1
                            end
                            
                     firstfloor_opendoor : begin
                           if(req_FF || req_E2_S2F) begin
                             E1_next_pos = secondfloor_closeddoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(req_SF || req_E1_F2S) begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(req_E1_S2F) begin
                             E1_next_pos = moving_second2first;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(!obs_sensor2 && open_timer2 && req_E2_F2S) begin
                             E1_next_pos = secondfloor_closeddoor;  
                             E2_next_pos = moving_first2second;  end 
                           else if(!obs_sensor2 && open_timer2 ) begin
                             E1_next_pos = secondfloor_closeddoor;  
                             E2_next_pos = firstfloor_closedoor;  end // 20
                           else begin
                             E1_next_pos = E1_pos;  
                             E2_next_pos = E2_pos;  end // 20.1
                           end
                           
                     moving_first2second  : begin
                            if(req_FF || req_E1_S2F) begin
                              E1_next_pos = moving_second2first;  
                              E2_next_pos = secondfloor_opendoor;  end
                            else if(req_SF || req_E1_F2S) begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = secondfloor_opendoor;  end
                            else if(req_E2_F2S || req_E2_S2F) begin
                              E1_next_pos = secondfloor_closeddoor;  
                              E2_next_pos = secondfloor_opendoor;  end // 21
                            else begin
                              E1_next_pos = E1_pos;  
                              E2_next_pos = secondfloor_opendoor;  end // 21.1
                            end
                            
                     secondfloor_closeddoor : begin
                              if(req_FF || req_E1_S2F) begin
                                E1_next_pos = moving_second2first;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(req_SF || req_E2_F2S) begin
                                E1_next_pos = secondfloor_closeddoor;  
                                E2_next_pos = secondfloor_opendoor;  end
                              else if(req_E1_F2S) begin
                                E1_next_pos = secondfloor_opendoor;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(req_E2_S2F) begin
                                E1_next_pos = secondfloor_closeddoor;  
                                E2_next_pos = moving_second2first;  end // 22
                              else begin
                                E1_next_pos = E1_pos;  
                                E2_next_pos = E2_pos;  end // 22.1
                              end
                              
                     secondfloor_opendoor  : begin
                             if(req_FF || req_E1_S2F) begin
                               E1_next_pos = moving_second2first;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(req_SF || req_E2_F2S) begin
                               E1_next_pos = secondfloor_closeddoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(req_E1_F2S) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(!obs_sensor2 && open_timer2 && req_E2_S2F) begin
                               E1_next_pos = secondfloor_closeddoor;  
                               E2_next_pos = moving_second2first;  end 
                             else if(!obs_sensor2 && open_timer2 ) begin
                               E1_next_pos = secondfloor_closeddoor;  
                               E2_next_pos = secondfloor_closeddoor;  end // 23
                             else begin
                               E1_next_pos = E1_pos;  
                               E2_next_pos = E2_pos;  end // 23.1
                             end
                             
                     moving_second2first : begin
                           if(req_FF || req_E2_F2S || req_E2_S2F) begin
                              E1_next_pos = secondfloor_closeddoor;  
                              E2_next_pos = firstfloor_opendoor;  end
                           else if(req_SF || req_E1_F2S) begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = firstfloor_opendoor;  end
                           else if(req_E1_S2F) begin
                              E1_next_pos = moving_second2first;  
                              E2_next_pos = firstfloor_opendoor;  end // 24
                           else begin
                             E1_next_pos = E1_pos;  
                             E2_next_pos = firstfloor_opendoor;  end // 24.1
                           end
                           
                     default : begin
                         E1_next_pos = E1_pos;
                         E2_next_pos = E2_pos;
                         end
                 endcase
            end
            
          secondfloor_opendoor  : begin
                case (E2_pos)
                     firstfloor_closedoor : begin
                            if(req_FF || req_E2_S2F) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = firstfloor_opendoor;  end
                            else if(req_SF || req_E1_F2S) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = firstfloor_closedoor;  end
                            else if(!obs_sensor1 && open_timer1 && req_E1_S2F) begin
                               E1_next_pos = moving_second2first;  
                               E2_next_pos = firstfloor_closedoor;  end
                            else if(req_E2_F2S) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = moving_first2second;  end 
                            else if(!obs_sensor1 && open_timer1 ) begin
                               E1_next_pos = secondfloor_closeddoor;  
                               E2_next_pos = firstfloor_closedoor;  end // 25
                             else begin
                               E1_next_pos = E1_pos;  
                               E2_next_pos = E2_pos;  end // 25.1
                            end
                            
                     firstfloor_opendoor : begin
                           if(req_FF || req_SF || req_E1_F2S || req_E2_S2F) begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(!obs_sensor1 && open_timer1 && req_E1_S2F) begin
                             E1_next_pos = moving_second2first;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(!obs_sensor2 && open_timer2 && req_E2_F2S) begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = moving_first2second;  end 
                           else if(!obs_sensor1 && open_timer1 ) begin
                             E1_next_pos = secondfloor_closeddoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(!obs_sensor2 && open_timer2 ) begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = firstfloor_closedoor;  end // 26
                           else begin
                             E1_next_pos = E1_pos;  
                             E2_next_pos = E2_pos;  end // 26.1
                           end
                           
                     moving_first2second  : begin
                            if((!obs_sensor1 && open_timer1) && (req_FF || req_E1_S2F)) begin
                              E1_next_pos = moving_second2first;  
                              E2_next_pos = secondfloor_opendoor;  end
                            else if(req_SF || req_E1_F2S || req_E2_F2S || req_E2_S2F) begin
                              E1_next_pos = secondfloor_opendoor;  
                              E2_next_pos = secondfloor_opendoor;  end
                            else if(!obs_sensor1 && open_timer1) begin
                              E1_next_pos = secondfloor_closeddoor;  
                              E2_next_pos = secondfloor_opendoor;  end // 27
                            else begin
                              E1_next_pos = E1_pos;  
                              E2_next_pos = secondfloor_opendoor;  end // 27.1
                            end
                            
                     secondfloor_closeddoor : begin
                              if(req_FF || req_E2_S2F) begin
                                E1_next_pos = secondfloor_opendoor;  
                                E2_next_pos = moving_second2first;  end
                              else if(req_SF || req_E1_F2S) begin
                                E1_next_pos = secondfloor_opendoor;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(!obs_sensor1 && open_timer1 && req_E1_S2F) begin
                                E1_next_pos = moving_second2first;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(req_E2_F2S) begin
                                E1_next_pos = secondfloor_opendoor;  
                                E2_next_pos = secondfloor_opendoor;  end 
                              else if(!obs_sensor1 && open_timer1 ) begin
                                E1_next_pos = secondfloor_closeddoor;  
                                E2_next_pos = secondfloor_closeddoor;  end // 28
                              else begin
                                E1_next_pos = E1_pos;  
                                E2_next_pos = E2_pos;  end // 28.1
                              end
                              
                     secondfloor_opendoor  : begin
                             if((!obs_sensor1 && open_timer1) && (req_FF || req_E1_S2F)) begin
                               E1_next_pos = moving_second2first;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(req_SF || req_E1_F2S || req_E2_F2S) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(!obs_sensor2 && open_timer2 && req_E2_S2F) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = moving_second2first;  end 
                             else if(!obs_sensor1 && open_timer1) begin
                               E1_next_pos = secondfloor_closeddoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(!obs_sensor2 && open_timer2) begin
                               E1_next_pos = secondfloor_opendoor;  
                               E2_next_pos = secondfloor_closeddoor;  end // 29
                             else begin
                               E1_next_pos = E1_pos;  
                               E2_next_pos = E2_pos;  end // 29.1
                             end
                             
                     moving_second2first : begin
                           if(req_FF || req_SF || req_E1_F2S || req_E2_F2S || req_E2_S2F) begin
                             E1_next_pos = secondfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if(!obs_sensor1 && open_timer1 && req_E1_S2F) begin
                             E1_next_pos = moving_second2first;  
                             E2_next_pos = firstfloor_opendoor;  end 
                           else if(!obs_sensor1 && open_timer1 ) begin
                             E1_next_pos = secondfloor_closeddoor;  
                             E2_next_pos = firstfloor_opendoor;  end // 30
                           else begin
                             E1_next_pos = E1_pos;  
                             E2_next_pos = firstfloor_opendoor;  end // 30.1
                           end
                           
                     default : begin
                         E1_next_pos = E1_pos;
                         E2_next_pos = E2_pos;
                         end
                 endcase
            end
            
          moving_second2first : begin
                case (E2_pos)
                     firstfloor_closedoor : begin
                            if(req_FF || req_E2_S2F) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = firstfloor_opendoor;  end
                            else if(req_SF || req_E2_F2S) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = moving_first2second;  end
                            else if(req_E1_F2S || req_E1_S2F) begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = firstfloor_closedoor;  end // 31
                            else begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = E2_pos;  end // 31.1
                            end
                            
                     firstfloor_opendoor : begin
                           if(req_FF || req_E1_F2S || req_E1_S2F || req_E2_S2F) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  end
                           else if((!obs_sensor2 && open_timer2) && (req_SF || req_E2_F2S)) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = moving_first2second;  end 
                           else if(!obs_sensor2 && open_timer2) begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = secondfloor_closeddoor;  end // 32
                           else begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = E2_pos;  end // 32.1
                           end
                           
                     moving_first2second  : begin
                              E1_next_pos = firstfloor_opendoor;  
                              E2_next_pos = secondfloor_opendoor;  /// 33
                            end
                            
                     secondfloor_closeddoor : begin
                              if(req_FF || req_E1_F2S || req_E1_S2F) begin
                                E1_next_pos = firstfloor_opendoor;  
                                E2_next_pos = secondfloor_closeddoor;  end
                              else if(req_SF || req_E2_F2S) begin
                                E1_next_pos = firstfloor_opendoor;  
                                E2_next_pos = secondfloor_opendoor;  end
                              else if(req_E2_S2F) begin
                                E1_next_pos = firstfloor_opendoor;  
                                E2_next_pos = moving_second2first;  end // 34
                              else begin
                                E1_next_pos = firstfloor_opendoor;  
                                E2_next_pos = E2_pos;  end // 34.1
                              end
                              
                     secondfloor_opendoor  : begin
                             if(req_FF || req_SF || req_E1_F2S || req_E1_S2F || req_E2_F2S) begin
                               E1_next_pos = firstfloor_opendoor;  
                               E2_next_pos = secondfloor_opendoor;  end
                             else if(!obs_sensor2 && open_timer2 && req_E2_S2F) begin
                               E1_next_pos = firstfloor_opendoor;  
                               E2_next_pos = moving_second2first;  end  
                             else if(!obs_sensor2 && open_timer2 ) begin
                               E1_next_pos = firstfloor_opendoor;  
                               E2_next_pos = secondfloor_closeddoor;  end  // 35
                             else begin
                               E1_next_pos = firstfloor_opendoor;  
                               E2_next_pos = E2_pos;  end // 35.1
                             end
                             
                     moving_second2first : begin
                             E1_next_pos = firstfloor_opendoor;  
                             E2_next_pos = firstfloor_opendoor;  /// 36
                           end
                           
                     default : begin
                         E1_next_pos = firstfloor_opendoor;
                         E2_next_pos = E2_pos;
                         end
                 endcase
            end
            
          default : begin
                    E1_next_pos = E1_pos;
                    E2_next_pos = E2_pos;
                    end
      endcase
          
     end

endmodule
