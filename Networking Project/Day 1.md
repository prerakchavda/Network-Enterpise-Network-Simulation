I went into the IO and I configured the terminal and enabled routing globally by putting in IP routing and then I went into each interface and edited them with a description I gave them a static IP address with their subnet being slash 30 so that only the quote unquote ISP and router have each AIP Which prevents then I did no shutdown on the ports to make sure that they are on
![[Editing Interfaces in IOU.png]]




This is where i made the loopback be 8.8.8.8 so that it simulates Google

![[Loopback Ip change.png]]



Then I hoped on NY-EDGE-1
![[Setting up edge routers.png]]
And configred the ip for int 1 to have the same one set in the IOU and did that for rest of the routers


made sure to test by pinging 8.8.8.8:
![[Ping test on 8.8.8.8 from edge router.png]]