## Golang graceful shutdown

### Usage: 
    ctx, cancel := context.WithCancel(context.Background())
	gsh := shutdown.NewGraceful(ctx, cancel)

    gateway, err := api.NewGateway(ctx, output, livenessProbe)
	if err != nil {
		return lgr.Fatal(ctx, errors.New("starting the api gateway failed"), logger.Fields{"err": err.Error()})
	}

	dispatcher, err := event.NewDispatcher(ctx, output, livenessProbe)
	if err != nil {
		return lgr.Fatal(ctx, errors.New("starting the event dispatcher failed"), logger.Fields{"err": err.Error()})
	}

	gsh.Add(1)
	go func() {
		defer gsh.Done()
		gateway.Start()
	}()

	gsh.Add(1)
	go func() {
		defer gsh.Done()
		dispatcher.Start()
	}()

	gsh.ListenCancelAndAwait()

	lgr.InfoMsg(ctx, "the entire app (gateway and dispatcher) has been successfully shut down, exiting", nil)