# Project Title
Versum

## Authors
Tarik Barri

## Description
Versum is a musical sequencer, designed as an audiovisual realtime virtual 3D world that invites both the audience and the composer to look at the music and listen to the visuals. It was born out of a twofold need: 
1.	A wish to tear down walls between the digital composer and the audience. I want to literally show people what my compositions look like. 
2.	A need for innovation in the practice of music creation: What if there were more than one possible way to move through compositions in time, in multiple dimensions?
 Versum is seen and heard from the viewpoint of a moving virtual camera, controlled in realtime, with virtual microphones attached. It moves past entities that can be both seen and heard. Like in reality, the closer the camera is to them, louder they are heard. Therefore, when moving past several visual objects, the camera’s movement causes sounds to fade in and out in a sequence determined by its path.

## Link to Prototype
No link to prototype.

## Example Code
The code is very complex already, with heaps and heaps of methods and classes, so here's just a tiny bit of it, which will not make much sense in itself. This code calls the functions necessary to complete main general drawing sequence, used for ever new frame
```
    static public void drawAll() {
        SlabChain.currentRenderLayer=0;
        GlTempShape.removeUnused();
        //---------------------------------//
        Verse.me.drawer.draw();
        Entity.bugFixSphere.setScreenPos(Verse.me, new Pos(-50,250,50));

        if (!(ConstList.screenConst.eHash.containsValue(bugFixSphere))) {
            ConstList.screenConst.addEntity(bugFixSphere);
        }   

        // go through each slabchain and throw to screen one at a time
        initialTexNo+=1;
        initialTexNo%=GL.initialTextures;
        SlabDest lastDest=null;
        for (int currentLayer=0; currentLayer<SlabChain.layers; currentLayer++) {
            // first aura's
            renderLayer=currentLayer;
            Renderer.setupRender((int)M.clip(1 - renderLayer,0,1),true); //  no idea why this is needed, seems completely illogical

            // DRAW AURAS
            for (int i = 0; i < LocEntity.sortedVisibleLocEntities.length; i++) {
                LocEntity le = LocEntity.sortedVisibleLocEntities[i];
                if (le.visible.getBoolVal()) {
                    if (le.drawLayer.getFVal()==currentLayer && 
                            le instanceof AuraEntity &&
                            le.updated) {
                        AuraEntity thisSE = (AuraEntity)LocEntity.sortedVisibleLocEntities[i];
                        if (thisSE!=bugFixSphere) {
                            thisSE.drawerAura.draw();
                        }
                    } else {
                        if (le.drawLayer.getFVal()==currentLayer && le instanceof TweetMap && le.updated) {
                            le.draw();
                        }
                    }
                }
            }
            // DRAW CORES
            for (int i = 0; i < LocEntity.sortedVisibleLocEntities.length; i++)
            {
                int j=i;
                LocEntity le = LocEntity.sortedVisibleLocEntities[j];
                if (le.visible.getBoolVal()) {
                    if (le.drawLayer.getFVal()==currentLayer &&
                            le.updated &&
                            !(le instanceof TweetMap)) {
                        le.draw(); // ??  drawing the bugfixsphere here makes all the difference for some reason
                    }
                }
            }
            
            if (Versum.useSlabChain) {
                SlabChain sc = SlabChain.getSlabChain(currentLayer);
                sc.slabThru.setGate(1); // standard: bypass slabthru
                sc.setLayer(currentLayer); // informing the slabchain to which layer it belongs; slab values will be updated to fit the layer
                int passes = sc.getPasses();
                SlabDest slabDest=null;
                sc.renderToInitialTexture(initialTexNo); // render to videoplane 0
                for (int pass=0; pass<passes; pass++) {
                    sc.updateSlabDynpars(pass);
                    slabDest=sc.getSlabDest(pass);
                    GL.lastSlab.setGateDestination(slabDest);
                    // send the texture through the slabs!
                    if (lastDest==null || lastDest==GL.lastSlab.feedbackDest1 || lastDest==GL.lastSlab.feedbackDest2 ||
                            lastDest==GL.lastSlab.vidPlaneDest) {
                        sc.sendInitialTextureToSlabs(initialTexNo); // this bangs on the rendered-to texture
                    } else {
                        if (lastDest.isStorage()){
                            lastDest.bang();
                        }
                    }
                    if (slabDest==GL.lastSlab.vidPlaneDest) {// to vidplane? then make sure vidplane replaces rendercontext
                        GL.eraseRenderContext();
                        GL.vidPlaneList.get(0).bang();
                    }                    
                    lastDest=slabDest;
                }
            }
        }
    }
    
```
## Links to External Libraries
Currently java, max/msp, max4live and ableton live are required to have Versum fully functional.

## Images & Videos
Video link: http://tarikbarri.nl/projects/versum
http://www.youtube.com/watch?v=NXinV7BsIbE
https://vimeo.com/81614819 password: versum

